# Scope Fence Rules

Loaded at Stage 2 when the boundary between in-scope and out-of-scope
is genuinely ambiguous. Apply these rules in order; stop at the first
that resolves the question.

---

## Rule 1 — Named beats implied

If the work item names a specific file, function, component, or endpoint,
only that named artifact (and the files it directly depends on to compile)
is in scope. Anything the change makes you *want* to refactor is out of scope.

## Rule 2 — Feature brief boundary

A feature brief is scoped to the described behavior and the smallest set
of files that implement it. UI, API handler, and unit tests for new code
are in scope. Refactoring pre-existing code is out of scope unless the
brief explicitly says "refactor X".

## Rule 3 — Review findings boundary

A review finding is scoped to the exact line(s) called out. Fixing a
similar pattern elsewhere in the codebase is out of scope unless the
finding says "fix all occurrences".

## Rule 4 — Five-file threshold

If implementing the work item as described requires touching more than
five files, stop and re-read the brief. Either:
- The brief is genuinely scoped to those files → proceed, noting the
  breadth in the PR description.
- The brief implies a structural change the team should approve first →
  implement only the surface-level portion that is unambiguous, and add
  a clear "requires architectural decision" note in the PR description.

## Rule 5 — Security observations

If you notice a security issue while implementing that is NOT covered by
the work item:
- If it is in a file you are already modifying AND it is a one-line fix
  (null check, input validation gap), include it and note it in the PR.
- If it requires its own thought and testing, add it to out-of-scope notes.

## Rule 6 — Opportunistic improvements

Never refactor, rename, reformat, or "clean up" code that is not directly
touched by the work item. Such changes inflate diff noise, make reviews
harder, and may introduce regressions outside the work item's intent.

## Rule 7 — Test additions

Tests for new code introduced by the work item are in scope.
Tests for pre-existing untested code are out of scope unless the brief
explicitly requests them.

## Rule 8 — Doubt → note, don't build

When in doubt whether something is in scope, add it to the out-of-scope
notes and let the human reviewer decide. The cost of an unnecessary note
is one extra line in the PR description. The cost of an unnecessary code
change is a review cycle.
