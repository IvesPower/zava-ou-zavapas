# CI recovery checklist

Consumed by: `SKILL.md` Step 5 (review-driver).

Every push from the review-driver MUST be followed by CI observation.
A push that is not observed green is not a landing candidate.

## Watch contract

```bash
gh pr checks <PR_NUMBER> --repo <REPO> --watch
```

`--watch` blocks until the check set is conclusive. If `--watch` is
unavailable, fall back to polling:

```bash
while true; do
  out=$(gh pr checks <PR_NUMBER> --repo <REPO>)
  echo "$out"
  echo "$out" | grep -qE '(pending|queued|in_progress)' || break
  sleep 30
done
```

Settle on one of: ALL GREEN, ANY FAIL.

## On ALL GREEN

Proceed to Step 6 (report completion). Record the green check
summary as `ci_evidence`.

## On ANY FAIL

Read the failing check log:

```bash
gh run view <run-id> --repo <REPO> --log-failed
```

Classify the failure into one of four buckets:

### Bucket 1 — Lint failure

Symptom: ESLint, Prettier, or TypeScript compiler errors on touched
files.

Recovery:
1. Run the linter locally: `npm run lint -- --fix`.
2. Run the formatter: `npx prettier --write <files>`.
3. Verify clean: `npm run lint && npx tsc --noEmit`.
4. Commit with a descriptive message.
5. Push. Re-enter watch.

### Bucket 2 — Test failure

Symptom: Jest or Vitest red in the failing job log.

Recovery:
1. Reproduce locally: `npm test -- --testPathPattern=<failing-test>`.
2. Read the trace, identify root cause.
3. If the test asserts on behavior this PR introduces, fix the
   production code. If the test is a pre-existing flake, fix the
   test with a clear comment.
4. Re-run the test until green.
5. Commit + push + re-enter watch.

### Bucket 3 — CI infra hiccup (transient)

Symptoms: network timeout, runner pre-empted, GitHub Actions service
disruption. Same job passed on the parent commit.

Recovery:
1. Re-run the failed job: `gh run rerun <run-id> --failed --repo <REPO>`.
2. Watch again.
3. Each run-id gets at most ONE re-run. A second failure is no longer
   transient — escalate to Bucket 4.

### Bucket 4 — Persistent unknown failure

Symptom: failure does not match buckets 1-3; same job fails twice.

Recovery:
1. Record the failing job name, run-id URL, and a 30-line excerpt of
   the failing log.
2. Report blocked. The summary comment names the failing job and
   points the maintainer at the run URL.

## Iteration cap

**Hard cap: 1 CI recovery iteration per review-driver run.** This is
the workshop-simplified version (the production shepherd-driver
allows 3). Beyond the cap the driver terminates with status: blocked.

## What flows back

The review-driver records in its completion:

```json
{
  "ci_iterations": 0,
  "ci_evidence": "URL of the final green run, or summary of the failing job"
}
```
