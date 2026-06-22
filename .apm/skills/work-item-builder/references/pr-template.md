# PR Body Template

Populate each section. Remove any section header whose content is empty.

---

## What this PR does

<!-- One bullet per file created or modified. Example:
- `app/components/Banner.tsx` — added free-shipping banner component
- `app/page.tsx` — renders Banner when cart total > 50
- `tests/banner.test.ts` — unit tests for Banner display logic
-->

- [LIST IN-SCOPE CHANGES HERE]

## Work item

<!-- Paste the first ~150 chars of the original work item or brief. -->

> [WORK ITEM SUMMARY]

## Quality checks

| Check | Result |
|---|---|
| `npm run lint` | PASS / FAIL (N errors) |
| `npm test` | PASS / FAIL (N failures) |

## Security checklist

From `secure-coding-base.instructions.md` section 8:

- [ ] No new secrets in diff (gitleaks clean)
- [ ] All new HTTP handlers have authN + authZ
- [ ] All new DB queries are parameterized
- [ ] Dependencies justified in PR description (if any added)
- [ ] Logs masked for PII

## Out-of-scope observations

The following were noticed but fall outside this work item's boundary.
A human reviewer should decide whether to act on these:

<!-- Leave blank if there are none — remove this section if empty. -->

- [LIST OUT-OF-SCOPE NOTES HERE]

---

*Opened by work-item-builder skill on behalf of Copilot.*
