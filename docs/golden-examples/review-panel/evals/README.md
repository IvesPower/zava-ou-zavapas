# review-panel evals

Two complementary eval types live here.

## 1. `trigger-evals.json` (dispatch description eval)

6 should-trigger + 6 should-NOT-trigger queries split 50/50 train/val.
The validation split is the ship gate: rate >= 0.5 on should-trigger
AND < 0.5 on should-NOT-trigger.

This is a manual eval against the dispatch description in SKILL.md's
frontmatter — run by reading the description as if you were the
harness's dispatcher LLM and classifying each query.

### What "passing" looks like

- Every `should_trigger.val` prompt clearly matches the description's
  trigger guidance (labelled PR, multi-persona review, panel).
- Every `should_not_trigger.val` prompt asks for something the panel
  does NOT do (single-file review, explanation, PR authoring, linting).

## 2. Schema validation (eval-as-stop-condition)

The panel's recursion is bounded by schema-validated persona returns.
A panelist that does not return valid JSON per
`assets/panelist-return-schema.json` is re-run, not trusted. This is
the eval-as-stop-condition discipline: the schema IS the eval.

To verify a panelist return offline:

```bash
python3 -c "
import json, jsonschema
schema = json.load(open('assets/panelist-return-schema.json'))
data = json.load(open('test-return.json'))
jsonschema.validate(data, schema)
print('PASS')
"
```

### Adding test fixtures

Drop `<scenario>.json` files into this directory with example panelist
returns. Each should conform to `assets/panelist-return-schema.json`.
Run the schema validation above to verify.
