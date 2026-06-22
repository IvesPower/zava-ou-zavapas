---
name: review-panel
description: >-
  Use this skill to run a multi-persona expert advisory review on a
  labelled pull request against zava-storefront. The panel fans out
  to three specialist reviewers (correctness, security, test-coverage),
  each running in its own agent thread, plus a synthesizer that
  arbitrates and renders ONE recommendation comment. Activate when a
  PR is labelled `panel-review` or when a non-trivial PR needs a
  cross-cutting recommendation before merge.
---

# Review Panel — Fan-Out Advisory Review

The panel is FAN-OUT + SYNTHESIZER. Each persona runs in its own agent
thread (via the `task` tool) and returns JSON matching
`assets/panelist-return-schema.json`. The orchestrator schema-validates
each return, hands all returns to the synthesizer, then renders ONE
recommendation comment from `assets/recommendation-template.md`.

This skill is ADVISORY by design. It does not compute a binary verdict,
it does not apply verdict labels, and it does not gate merge. The panel
surfaces findings; the maintainer and the PR author decide ship.

## Architecture invariants

- **Advisory regime, not gate regime.** There is no APPROVE / REJECT,
  no `panel-approved` / `panel-rejected` label, no deterministic verdict
  computation. The synthesizer returns a `stance` (`ship_now` /
  `ship_with_followups` / `needs_discussion` / `needs_rework`); this is
  prose for the human reviewer, never auto-applied as a label or check.
- **Three severity buckets, none of them gate.** Findings carry
  `severity: blocking | recommended | nit`. `blocking` is the highest
  signal a persona can send and renders prominently in the comment; it
  still does not block merge. `recommended` is the default for
  substantive feedback. `nit` is one-line polish.
- **Single-writer interlock.** Only the orchestrator writes to the PR:
  exactly one comment. Persona subagents and the synthesizer return
  JSON only and MUST NOT call any `gh` write command, post comments,
  apply labels, or touch the PR state.
- **Eval-as-stop-condition.** The panel's recursion is bounded in
  *space* by schema-validated persona returns — a panelist that does
  not return valid JSON matching `assets/panelist-return-schema.json`
  is re-run (max 1 retry), not trusted. It is bounded in *time* by a
  persisted plan note (see §Plan persistence below).

## Agent roster

| Persona | File | Lens |
|---|---|---|
| Correctness Reviewer | `personas/correctness-reviewer.md` | Logic, data integrity, breaking changes |
| Security Reviewer | `personas/security-reviewer.md` | Injection, auth, secrets, dependencies |
| Test Coverage Reviewer | `personas/test-coverage-reviewer.md` | Missing tests, thin coverage, test quality |
| Synthesizer | `personas/synthesizer.md` | Deduplication, prioritization, ship stance |

## Topology

```
   review-panel SKILL (orchestrator thread)
                      |
   FAN-OUT via task tool (3 personas in parallel)
                      |
   +------------------+------------------+
   v                  v                  v
 correctness      security         test-coverage
   |                  |                  |
   | each returns JSON per panelist-return-schema.json
   +------------------+------------------+
                      |
                      v   <-- schema-validate each return
                      v   <-- on malformed: re-spawn that persona (max 1 retry)
                      v
   task: synthesizer
   - deduplicates findings across personas
   - resolves dissent
   - emits headline + arbitration prose
   - emits curated follow-ups (max 5)
   - emits stance + recommendation prose
                      |
                      v
   orchestrator (sole writer)
            |
            v
        add-comment
        (exactly 1, rendered from
         assets/recommendation-template.md)
```

## Execution steps

### Step 1 — Preflight

1. Load the PR diff: `gh pr diff <PR_NUMBER> --repo <REPO>`.
2. Load the PR metadata: `gh pr view <PR_NUMBER> --repo <REPO> --json title,body,files`.
3. Record a persisted plan note: `"review-panel: PR #<N>, started <ISO timestamp>, max runtime 10 min"`.

### Step 2 — Fan-out to personas

For each persona in the roster, spawn a `task` subagent:

```
task(agent_type="explore", prompt="""
You are the {{ persona_name }}. Read personas/{{ persona_slug }}.md for
your role and inspection rules.

Here is the PR diff:
<diff>
{{ diff }}
</diff>

Here is the PR metadata:
<metadata>
{{ metadata }}
</metadata>

Return ONLY a JSON object matching assets/panelist-return-schema.json.
Set persona to "{{ persona_slug }}".
Do NOT post comments or write to the PR.
""")
```

All three `task` calls are made in parallel (independent fan-out).

### Step 3 — Schema validation

For each persona return:
1. Parse as JSON.
2. Validate against `assets/panelist-return-schema.json`.
3. If valid → accept.
4. If invalid → re-spawn that persona with the validation error
   appended to the prompt (max 1 retry). If the retry also fails,
   log an error and exclude that persona from the synthesis.

### Step 4 — Synthesizer

Spawn a `task` subagent with all validated persona returns:

```
task(agent_type="general-purpose", prompt="""
You are the synthesizer. Read personas/synthesizer.md for your role.

Here are the panelist returns:
<panelist_returns>
{{ JSON array of validated returns }}
</panelist_returns>

Deduplicate, prioritize, and emit a JSON object with: headline,
arbitration, follow_ups (max 5), stance, recommendation_prose.
Do NOT post comments or write to the PR.
""")
```

### Step 5 — Render and post

1. Load `assets/recommendation-template.md`.
2. Fill the template with synthesizer output + per-persona summary
   counts (count blocking/recommended/nit per persona from their
   validated returns).
3. Post exactly ONE comment on the PR:
   `gh pr comment <PR_NUMBER> --repo <REPO> --body "<rendered>"`.
4. Remove the trigger label to prevent re-runs:
   `gh pr edit <PR_NUMBER> --repo <REPO> --remove-label panel-review`.

### Step 6 — Close plan note

Update the persisted plan note: `"review-panel: PR #<N>, completed <ISO timestamp>, stance: <stance>"`.

## Plan persistence

The orchestrator writes a plan note at start and updates it at
completion. This bounds the panel's runtime: if the plan note is
older than 10 minutes with no completion update, the outer
orchestrator (review-driver or a CI workflow) treats the run as
timed out and reports accordingly.

## Bundled assets

- `personas/correctness-reviewer.md` — correctness lens.
- `personas/security-reviewer.md` — security lens.
- `personas/test-coverage-reviewer.md` — test coverage lens.
- `personas/synthesizer.md` — synthesis + ship recommendation.
- `assets/panelist-return-schema.json` — JSON schema for persona returns.
- `assets/recommendation-template.md` — comment template.
- `evals/trigger-evals.json` — dispatch description eval set.
- `evals/README.md` — eval documentation.
