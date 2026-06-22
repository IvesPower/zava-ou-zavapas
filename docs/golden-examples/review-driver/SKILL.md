---
name: review-driver
description: >-
  Use this skill to drive a pull request from review to green CI. It
  composes the review-panel as its per-PR review primitive, folds
  non-blocking findings as commits, pushes, and watches CI to green.
  Activate when a PR needs end-to-end review-and-fix before merge.
  This is the advanced driver skill from the Agentic SDLC track.
---

# Review Driver — 2-Step Review→Drive Orchestrator

The review-driver **composes** the
`review-panel` as its review step and drives a PR from "needs
review" to "CI green, ready to merge" in one automated pass.

This skill demonstrates the **composition mechanic** at the heart of
the Agentic SDLC track: the driver declares a dependency edge on
`review-panel` in its `apm.yml`, the lockfile pins it, and
`apm install` deploys the closure. A consumer who pins
`review-driver` gets the whole composed graph — panel personas,
schemas, templates — without knowing the internals. This is ch22's
"lockfile-pinned skill dependency graph" made concrete.

## Shape (distilled from `awd-cli/shepherd-driver`)

1. **Run the review-panel** on the target PR (the panel is the per-PR
   review primitive).
2. **Fold-by-default** — apply `recommended` and `nit` findings as
   commits using `assets/fold-vs-defer-rubric.md`. DEFER `blocking`
   findings to the human with an explicit scope-boundary note.
3. **Push** the fold commits to the PR's head branch.
4. **Watch CI to green** — poll the checks; on red, read the failure
   and attempt ONE bounded recovery pass per
   `assets/ci-recovery-checklist.md`. Report completion.

## Architecture invariants

- **Composition, not reimplementation.** The driver COMPOSES the
  review-panel; it does NOT reimplement panel review. The panel runs
  as a self-contained skill invocation that posts its own advisory
  comment. The driver reads the synthesizer output to decide what to
  fold.
- **Fold-by-default.** The axis is scope-creep risk, not severity.
  Anything that raises the quality bar of the PR's stated scope gets
  folded. Only items that introduce a new theme unrelated to the
  stated scope are deferred. See `assets/fold-vs-defer-rubric.md`.
- **Blocking findings are DEFERRED, not auto-fixed.** The driver
  treats `blocking` severity as a signal that needs human judgment.
  It does not attempt to auto-fix blocking findings. Each deferral
  includes a one-line boundary note.
- **CI-observed-green is the terminal condition.** The driver does not
  trust assumptions about CI. It polls `gh pr checks` and only
  reports success when all checks pass.
- **Bounded recovery.** On CI failure, the driver follows
  `assets/ci-recovery-checklist.md`: classify the failure (lint /
  test / infra / unknown), attempt ONE fix, re-push, re-watch. If
  the second run also fails, report blocked and stop.

## Execution steps

### Step 0 — Preflight

1. Verify the review-panel dependency is deployed:
   ```bash
   test -f ../review-panel/SKILL.md \
     && test -f ../review-panel/assets/panelist-return-schema.json \
     && echo "review-panel present" \
     || echo "MISSING review-panel - stop and report"
   ```
   On a probe MISS, stop and report blocked. Do not reimplement the
   panel inline.

2. Load the PR metadata:
   `gh pr view <PR_NUMBER> --repo <REPO> --json title,body,headRefName,number`.

3. Record a persisted plan note:
   `"review-driver: PR #<N>, started <ISO timestamp>"`.

### Step 1 — Run the review-panel

Invoke the review-panel skill on the target PR. The panel:
- Fans out to 3 specialist personas in parallel.
- Schema-validates each return.
- Runs the synthesizer.
- Posts ONE advisory comment on the PR.
- Returns the synthesizer output (stance + follow-ups).

Read the synthesizer output to proceed.

### Step 2 — Classify findings: fold vs defer

For each finding across all persona returns, apply the fold-vs-defer
rubric (`assets/fold-vs-defer-rubric.md`):

```
Is the finding already addressed in the latest commit?
  yes → skip (already resolved).

Does the finding touch code the PR already modifies, or raise the
quality bar of the PR's stated scope?
  yes → FOLD.

Does the finding introduce a new theme unrelated to the stated scope?
  yes → DEFER with a scope_boundary_crossed note.

Is the finding severity=blocking?
  yes → DEFER (needs human judgment).

Default → FOLD.
```

### Step 3 — Apply fold commits

For each FOLD item:
1. Read the relevant file(s) from the PR branch.
2. Apply the fix (code change, test addition, doc update).
3. Stage and commit with a descriptive message referencing the
   persona and finding.

### Step 4 — Push

```bash
git push origin HEAD
```

If push fails (force-push protection, permissions), report blocked.

### Step 5 — Watch CI to green

Follow `assets/ci-recovery-checklist.md`:

1. Poll checks: `gh pr checks <PR_NUMBER> --repo <REPO> --watch`.
2. **On ALL GREEN** → proceed to Step 6.
3. **On ANY FAIL** → classify the failure:
   - **Lint failure** → auto-fix with linter, commit, push, re-watch.
   - **Test failure** → reproduce locally, fix, commit, push, re-watch.
   - **Infra hiccup** → re-run the failed job once.
   - **Unknown** → attempt one fix based on the failure log.
4. If the second CI run also fails → report blocked with the failing
   job name and log excerpt.

**Hard cap: 1 CI recovery iteration.** This is the workshop-simplified
version; the production shepherd-driver allows 3.

### Step 6 — Report completion

Update the persisted plan note:
`"review-driver: PR #<N>, completed <ISO timestamp>, status: <status>"`.

Post a summary comment on the PR:
- What findings were folded (count + brief list).
- What findings were deferred (count + boundary notes).
- CI status (green / blocked).
- The panel's ship stance for reference.

## Bundled assets

- `assets/fold-vs-defer-rubric.md` — the fold-by-default decision
  rubric.
- `assets/ci-recovery-checklist.md` — bounded CI diagnosis + recovery.
- `apm.yml` — manifest with the `review-panel` dependency edge.

## Composition mechanic (the lesson)

The `apm.yml` in this directory declares:

```yaml
dependencies:
  apm:
    - ../review-panel
```

This is a **local-path dependency edge**. When a consumer runs
`apm install` on the review-driver, the resolver follows this edge
and deploys the review-panel alongside it. The **lockfile**
(`apm.lock.yaml`) pins the exact version of review-panel that was
tested with this driver, so consumers get a reproducible closure.

This is ch22's "lockfile-pinned skill dependency graph" — the same
mechanism that makes `npm install express` deploy express + its 57
transitive dependencies at pinned versions. The workshop participant
builds this edge by hand, experiencing the composition primitive that
the production factory uses at scale.

## Theory anchors

- ch22: The Reference Architecture Earned — lockfile-pinned skill graph.
- ch17: Multi-Agent Orchestration — fan-out + synthesizer pattern.
- ch18: The Execution Meta-Process — plan persistence bounds recursion.
