# Fold-vs-defer rubric

Consumed by: `SKILL.md` Step 2 (review-driver).

The review-driver uses this rubric to decide, for each finding
surfaced by the review-panel, whether to FOLD it into THIS PR or
DEFER it to the human.

## The default is FOLD

The old default was "post advisory; address later." That model leaves
PRs landing with known shortfalls the next contributor inherits. The
new default is FOLD: the PR lands in the best shape it can without
changing what the PR is ABOUT.

## The decision axis

The axis is NOT severity. The axis is SCOPE-CREEP RISK relative to
the PR's stated scope.

> Stated scope = the one-sentence description in the PR title and the
> first paragraph of the PR body.

### FOLD when the finding raises the quality bar of the stated scope

Examples that MUST be folded:

- Missing tests for behavior THIS PR introduces.
- Documentation drift caused BY this change.
- Security hardening on the new code path (input validation on a
  newly added API route).
- Naming consistency on new symbols.
- Lint failures on touched files.

### DEFER only when the finding introduces a new theme unrelated to the stated scope

Examples that legitimately defer:

- PR fixes a cart calculation bug; reviewer recommends a wholesale
  checkout-flow refactor across the codebase.
- PR adds a new API route; reviewer recommends migrating the entire
  auth middleware to a different pattern.
- PR updates a component; reviewer recommends a cross-cutting design
  system overhaul.

Every deferral MUST include a one-line `scope_boundary_crossed` note
naming the boundary.

### ALWAYS DEFER blocking findings

`blocking` severity means the finding needs human judgment — a
correctness regression, security bypass, or architectural fault. The
driver does not have enough context to auto-fix these safely. Defer
with an explicit note explaining why human review is needed.

## Anti-reasons for deferral (NEVER use these)

- "It's just a recommended item, not blocking." Severity is not the
  axis for recommended/nit items.
- "The PR is already big." If the in-scope follow-up is small, the
  PR being big does not change the fold decision.
- "The reviewer can address this in a follow-up PR." The whole point
  of this rubric is to not push the cost forward.

## Quick decision tree

```
Is the item already addressed in the latest commit?
  yes → skip (already resolved).

Is the item severity=blocking?
  yes → DEFER (needs human judgment).

Does the item touch code that THIS PR's diff already modifies, or
extend the contract THIS PR introduces?
  yes → FOLD.

Does the item raise the quality bar of the stated scope (tests,
docs, security, naming, lint)?
  yes → FOLD.

Does the item introduce a new theme unrelated to the stated scope?
  yes → DEFER with a scope_boundary_crossed note.

Default → FOLD.
```

When in doubt: FOLD. The cost of an extra small fold is bounded; the
cost of a missed fold compounds over future PRs.
