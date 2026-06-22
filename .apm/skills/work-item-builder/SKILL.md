---
name: work-item-builder
description: >-
  Use this skill when the user wants to implement a feature, address review
  findings, or turn a work item into committed code and a pull request on
  zava-storefront. Fires when you see: "implement this feature", "address
  these review comments", "open a PR for this change", "build this", "turn
  this brief into code", "fix these findings before merge", or when handed
  a list of TODO items / review notes and asked to commit the result.
  Creates a branch, reads team guidelines, plans the change, implements it,
  runs lint + tests, fixes failures within a bounded retry budget, then
  opens or updates the PR. Out-of-scope observations are noted in the PR
  description for a human — this skill never exceeds the stated boundary.
  Does NOT review, approve, or merge PRs (use panel-review). Does NOT
  process incident postmortems (use incident-to-pr). Does NOT migrate
  frameworks (use framework-modernizer).
license: UNLICENSED
allowed-tools: Read, Grep, Glob, Edit, Bash
---

# work-item-builder

Implements a scoped change on a new branch, runs the project's quality
checks, and opens (or updates) a pull request on zava-storefront.

---

## Prerequisites

Before starting implementation:

1. Plan commit boundaries early. Do **not** make one monolithic commit.
2. Split by concern so review stays readable:
   - tests (`test:`) in their own commit
   - docs (`docs:`) in their own commit
   - feature/fix code (`feat:` / `fix:`) in its own commit
   - optional setup/chore (`chore:`) in its own commit
3. Keep the final user-facing message minimal: outcome, PR link, blockers only.

---

## Attention anchor

Reload this block at the start of every stage:

> **GOAL:** implement exactly what the work item asks — no more, no less.
> **GATE:** lint and tests must pass before the PR opens.
> **FENCE:** anything outside the work item goes into PR notes, not into code.

---

## Stage 1 — Ground: load guidelines

Before writing a single line of code, read each of these files so the
implementation already conforms to team standards:

```
.github/guidelines/security.md
.github/guidelines/architecture.md
.github/guidelines/documentation.md
```

Then also read the persona files for their deeper reasoning lenses:

```
.github/agents/architect.agent.md
.github/agents/security.agent.md
```

The instruction files (secure-coding-base, docs-style-guide) are
auto-loaded by the harness on path match; the guidelines above are the
storefront-specific distillation to keep in sharp focus while coding.

---

## Stage 2 — Classify and scope the work item

Classify the input as one of:
- **FEATURE BRIEF** — describes new behavior to build.
- **REVIEW FINDINGS** — a list of specific issues on existing code to fix.

Then apply the scope fence:

1. List every change the work item explicitly requests. These are **in scope**.
2. List anything you notice that the work item does NOT mention. These are
   **out of scope** — record them as notes; do not implement them.
3. If the work item implies changes spanning more than ~5 files or a
   structural redesign, split: implement what fits the brief, note the rest.

If the boundary is genuinely ambiguous, read `references/scope-fence-rules.md`.

Persist the scope decision (in-scope list + out-of-scope notes) to `plan.md`
before moving on.

---

## Stage 3 — Plan

Write a step-by-step implementation plan to `plan.md`. Include:

- Each file to create or modify (exact relative path from repo root).
- A one-line description of the change per file.
- The order in which changes depend on each other.

Reload `plan.md` at the start of each subsequent stage.

---

## Stage 4 — Create branch

First confirm the working tree is clean:

```bash
git -C <storefront-root> status --porcelain
```

If output is non-empty, stop and report to the user — do not proceed with
uncommitted changes from a prior session.

Create the branch:

```bash
git -C <storefront-root> checkout -b <type>/<short-kebab-slug>
```

Where `<type>` is one of `feat`, `fix`, `chore`, `docs`.
Example: `feat/add-discount-banner`.

---

## Stage 5 — Implement

Follow `plan.md` step by step. For each change:

- Use the harness file-edit tool (deterministic writes).
- Apply the architectural and security lens loaded in Stage 1.
- Do not add logic outside the planned changes.
- After every three file changes, reload `plan.md` before continuing.

If a planned change requires touching something clearly outside the original
scope, add it to the out-of-scope notes and skip it rather than expanding scope.

---

## Stage 6 — Check gate

Run both checks from the storefront root. Replace `<storefront-root>` with
the repo directory (e.g. `zava-storefront/`):

```bash
npm --prefix <storefront-root> run lint
```

```bash
npm --prefix <storefront-root> test
```

**Both pass →** proceed to Stage 7.

**Either fails →**

1. Load the full error output.
2. Identify the root cause from the error text (do not guess).
3. Apply a targeted fix.
4. Re-run the failing check.
5. **Retry budget: 2 attempts maximum.**
   - Checks pass on retry → proceed to Stage 7.
   - Checks still red after 2 retries → **stop**. Report the failure to the
     user with the exact error text and your diagnosis. Do NOT open a PR.

---

## Stage 7 — Commit and push

Stage and commit in **separate commits by concern** (never one big commit):

```bash
git -C <storefront-root> add tests/*
git -C <storefront-root> commit -m "test: <short imperative description>"

git -C <storefront-root> add README.md docs/*
git -C <storefront-root> commit -m "docs: <short imperative description>"

git -C <storefront-root> add app/ lib/
git -C <storefront-root> commit -m "<feat|fix>: <short imperative description>

<one-paragraph body: what changed and why>

Work-item: <first 72 chars of the original work item text>
Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>"
```

If one category has no changes, skip that commit.

Push:

```bash
git -C <storefront-root> push -u origin <branch-name>
```

---

## Stage 8 — Open or update PR

Build the PR body first. Read `references/pr-template.md` and populate it
with: (a) the in-scope changes from Stage 2, (b) lint + test outcomes from
Stage 6, (c) the out-of-scope notes from Stage 2, (d) the security
checklist from `secure-coding-base.instructions.md` section 8.

Write the body to a temp file:

```bash
BODY_FILE=$(mktemp /tmp/pr-body-XXXXXX.md)
# write the populated template into $BODY_FILE
```

Check whether a PR for this branch already exists:

```bash
gh pr list --head <branch-name> --json number,title
```

**No existing PR →**

```bash
gh pr create \
  --title "<type>: <short description>" \
  --body-file "$BODY_FILE" \
  --draft
```

**PR already exists →**

```bash
gh pr edit <number> --body-file "$BODY_FILE"
```

Report the PR URL to the user along with a one-line summary of the
out-of-scope notes (if any).

Response style for the user:
- 2-4 short lines max
- no recap of internal steps
- no optional suggestions unless blocked

---

## Reference documents

Load these only when the stated trigger fires — do not load eagerly.

| Document | Load when |
|---|---|
| `references/scope-fence-rules.md` | Stage 2: boundary between in-scope and out-of-scope is unclear |
| `references/pr-template.md` | Stage 8: building the PR body |
