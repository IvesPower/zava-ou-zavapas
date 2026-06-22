---
name: pr-review
description: Use this skill when the user asks to review, check, or assess a pull request -- or when they want to know if a PR is ready to merge, safe to ship, or aligned with team standards. Triggers on PR number mentions, diff evaluations, or code change assessments. Also activates when the user asks about security problems, architecture issues, or documentation gaps in a PR even without naming those dimensions. Reviews the diff against three team guideline files (security, architecture, documentation), fans out to three independent reviewer lenses, synthesises findings, and emits: MERGE or REQUEST CHANGES, the reason, then findings per dimension ordered by criticality.
---

## Goal (B8 attention anchor -- re-read at every step boundary)

Produce a structured PR review verdict: **MERGE** or **REQUEST CHANGES**,
followed by the reason and a findings table per dimension sorted by criticality
(BLOCKER > HIGH > MEDIUM > LOW).

This is the acceptance criterion. Do not emit partial output. Do not skip
dimensions. Do not alter the verdict template.

---

## Step 1 -- Read plan and identify PR

Note the goal above. Identify the PR number from the user message or
current session context. If no PR number is present, ask the user for it
before continuing.

---

## Step 2 -- Fetch PR data (S7 deterministic tool bridge)

Run both commands. Do not skip or guess the output.

```
gh pr view <N> --json number,title,author,baseRefName,headRefName
gh pr diff <N>
```

Save in session state (B4 plan memento):
- PR number, title, author
- Full diff text (referenced as `diff_content` in spawn prompts below)

If `gh` is unavailable or returns an error, surface it to the user and stop.

---

## Step 3 -- Fan out to three review lenses (A1 PANEL)

Spawn all three tasks **before waiting for any result**. Each spawn receives
the full diff as its payload.

### Spawn 1 -- Security lens (`agent_type: code-review`)

```
ROLE: security reviewer. RESPOND CAVEMAN.
READ: .github/guidelines/security.md
SCAN: diff below.
FIND: auth-bypass, unvalidated input, SQL string interpolation, secrets in
code, float money field, unprotected mutation route, PII in logs, error
details leaked to client response.
ANCHOR: blocker = exploitable auth-bypass or unprotected mutation endpoint
in prod path only.
PRESERVE EXACT: file paths, line numbers.
ESCAPE TO NORMAL: if finding involves ambiguous destructive action.
OUTPUT JSON ONLY:
{"findings":[{"sev":"blocker|high|medium|low","title":"...","file":"...","line":<int>,"fix":"<=20 words"}]}
NO PROSE.

DIFF:
<diff_content>
```

### Spawn 2 -- Architecture lens (`agent_type: code-review`)

```
ROLE: architecture reviewer. RESPOND CAVEMAN.
READ: .github/guidelines/architecture.md
SCAN: diff below.
FIND: lib/ importing next/*, business logic inside route handler, server
component fetching own API route over HTTP, db not injected as parameter,
Zod schema defined inside route handler instead of lib/, new external
dependency without justification comment, test importing from app/.
ANCHOR: blocker = violates an explicit must-not rule in the guidelines.
PRESERVE EXACT: file paths, line numbers, rule numbers.
OUTPUT JSON ONLY:
{"findings":[{"sev":"blocker|high|medium|low","title":"...","file":"...","line":<int>,"fix":"<=20 words"}]}
NO PROSE.

DIFF:
<diff_content>
```

### Spawn 3 -- Documentation lens (`agent_type: explore`)

```
ROLE: doc checker.
READ: .github/guidelines/documentation.md
SCAN: diff.
CHECK: new exported lib/ function without JSDoc, new route.ts without
3-line header comment, non-obvious business rule without inline comment,
new env var not in README, user-visible change without README update,
Zod schema not adjacent to its type, test name restates inputs instead
of expected behaviour.
OUTPUT JSON ONLY:
{"findings":[{"sev":"blocker|high|medium|low","title":"...","file":"...","line":<int>,"fix":"<=20 words"}]}
```

---

## Step 4 -- Fan in and synthesize

Wait for all three spawns to return. Collect their JSON receipts.

Re-read the goal (B8): output must be MERGE or REQUEST CHANGES + reason +
findings tables.

Spawn one synthesis task (`agent_type: general-purpose`) using the prompt
in `assets/synthesizer-prompt.md`, substituting:
- `<number>`, `<title>`, `<author>` from Step 2
- `<security_json>`, `<architecture_json>`, `<documentation_json>` from the
  three receipts above

---

## Step 5 -- Emit verdict

Return the synthesizer output verbatim to the user. Do not summarize,
reformat, or add commentary.

---

## Output template (for reference; synthesizer fills this)

```
## Verdict: MERGE | REQUEST CHANGES

**Reason:** <1-3 sentences. REQUEST CHANGES if any blocker or high finding
exists in any dimension.>

## Findings

### Security
| Severity | Finding | File | Line | Suggested fix |
|----------|---------|------|------|---------------|

### Architecture
| Severity | Finding | File | Line | Suggested fix |
|----------|---------|------|------|---------------|

### Documentation
| Severity | Finding | File | Line | Suggested fix |
|----------|---------|------|------|---------------|
```

Rules the synthesizer must follow:
- Sort each table: blocker first, then high, medium, low.
- If a dimension has no findings, write "No findings."
- Verdict is MERGE only when ALL dimensions have zero blocker and zero high findings.
- Do not invent findings. Synthesize only from the JSON receipts.
