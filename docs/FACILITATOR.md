# Facilitator guide

This guide is for the person running the Zava Skills Workshop for a room of up to ~50 software engineers.

You are not teaching one fixed path. You are supporting a mixed room where everyone follows the same five-step spine:

1. **Design** — use Genesis to shape the skill before writing `SKILL.md`.
2. **Build** — generate the skill from the design and review the output.
3. **Validate** — run it locally against `zava-storefront/`.
4. **Ship to CI** — package with `apm`, tag a release, and run it with `gh aw`.
5. **Consume** — pin the released skill from another repo.

The tracks differ in ambition, not in the payoff. Every track ends with a packaged skill running in CI.

---

## 🗓️ Shape of the day

Plan for a full day with setup, three track blocks, CI time, and a short wrap-up.

| Block | Time | What you do |
|---|---:|---|
| Room setup | 15 min | Confirm `apm install`, `zava-storefront/`, `gh`, `gh aw`, and harness access. |
| Track choice | 5 min | Help people pick the right track. Recommend Basics → `test-improver` for first-timers. |
| Main lab | 90–120 min | Participants work through their track guide. TAs watch for setup and CI blockers. |
| CI + consume | 30–45 min | Package, tag, release, trigger the `gh aw` workflow, then pin the skill elsewhere. |
| Share-out | 15–20 min | Ask: What did the skill do locally? What changed when it ran in CI? |

The room can be mixed:

- 🧰 **Basics** (~90 min) — pick one of `test-improver`, `docs-generator`, or `dependency-auditor`. This is the cleanest loop. `test-improver` is the safest default because `npm test` is the oracle.
- 🔄 **Framework Modernizer** (~90 min) — build a single-skill pipeline for a real Next 14 → 15 migration of `zava-storefront/`.
- 🏗️ **Agentic SDLC** (~145 min) — compose three skills into a build → review loop: a builder that implements a change and opens a PR, a review panel that reviews it, and a driver that runs a deterministic gate (lint + tests + coverage) then the panel until the PR is clean. Then release the loop, pin it from another repo, and run the panel in CI on a `panel-review` label. Use this for the strongest engineers or people who already understand the Basics loop.
- 🎁 **PROVISION** — optional bonus lab for early finishers, not a fourth track.

Your job is to keep everyone moving through the same five steps, even when their track content differs.

---

## ✅ Before the day: deployer setup truths

The workshop needs the org provisioned through [`DevExpGbb/zava-workshop-kit`](https://github.com/DevExpGbb/zava-workshop-kit). Do this before participants arrive.

### Org secrets for CI agent runs

Two fine-grained PATs gate the CI agent path. Set both as **organization Actions secrets** with `--visibility=all` because participant repos and forks need to see them.

| Secret | Purpose | Required shape |
|---|---|---|
| `COPILOT_GITHUB_TOKEN` | Lets `gh aw` use `engine: copilot` in Actions. | Fine-grained PAT. Resource owner is a Copilot-seated **user** account. Single permission: **Account → Copilot Requests: Read**. |
| `GH_AW_PLUGINS_TOKEN` | Lets workflows fetch packaged skill dependencies from GitHub Packages. | Fine-grained PAT with **Packages: Read**. |

Set/check them with:

```bash
gh secret list --org <org>
# org admins set them with --visibility=all
```

If a CI agent run fails with `Resource not accessible by personal access token` or `401 no token`, assume `COPILOT_GITHUB_TOKEN` is missing or not visible to the repo until proven otherwise.

### Demo trigger labels

Create these labels before the live demo if you use the reference CI flows:

| Label | Where | What it triggers |
|---|---|---|
| `panel-review` | Storefront PR | PR review panel. |
| `status/needs-triage` | Issue | Triage panel. |

For Basics tracks, the participant guides use track-specific labels such as `run-test-improver`, `run-docs-generator`, and `run-dependency-auditor`. Those can be created during the lab, but pre-creating them saves time.

---

## ⚠️ Setup gotchas to pre-empt

Say these out loud during setup:

- `gh skill` now ships inside `gh`. Do **not** install a separate `gh-skill` extension.
- `npm test --prefix zava-storefront` is the `test-improver` oracle. It is a track validation step, not global setup for everyone.
- `npm audit` exits non-zero when it finds vulnerabilities. For `dependency-auditor`, that is expected. Parse JSON; do not treat the exit code as the result.
- `zava-storefront/` is cloned beside this repo and is gitignored. If someone cannot find it, have them rerun the README setup commands.
- If `/genesis` does not load in a harness, rerun `apm install` and check `.agents/skills/genesis/` exists.
- CI failures around Copilot auth are usually org-secret visibility, not participant mistakes.

Keep the local and CI loops separate:

- **Local validation** does not need the org PATs.
- **CI agent runs** do need the org PATs.
- If the PATs are not ready, still run the workshop locally and show CI from a reference repo.

---

## 🔍 Preflight checklist

Run this the day before and again 30 minutes before the room starts, from a fresh clone of a participant repo.

```bash
apm install

git clone --branch workshop-v1 --depth 1 \
  https://github.com/DevExpGbb/zava-storefront.git zava-storefront
npm install --prefix zava-storefront

apm deps list
ls .agents/skills/genesis

gh auth status
gh repo view
gh aw --help >/dev/null
```

For the `test-improver` demo only, also run:

```bash
npm test --prefix zava-storefront
```

For the Framework Modernizer reference check, run the existing catalog regression if present:

```bash
node .apm/skills/framework-modernizer/evals/run.js
```

If a preflight command fails, fix it before the room starts or decide to run that part from a reference path.

---

## 🧰 Track notes: Basics

Participants pick one skill:

- `docs/tracks/01-test-improver.md`
- `docs/tracks/02-docs-generator.md`
- `docs/tracks/03-dependency-auditor.md`

### Done looks like

- A skill exists under `.apm/skills/<skill-name>/SKILL.md`.
- The participant validated it locally against `zava-storefront/`.
- `apm pack --archive` produces a bundle.
- A tag/release exists.
- A `gh aw` workflow runs the released skill on a labeled PR.
- Another repo can pin the skill in `apm.yml` and run `apm install`.

### Common stuck points

- People start editing `SKILL.md` before reading Genesis's design. Stop them and make them compare the output to the design diagram.
- `test-improver` uses Vitest, not Jest. `npm test --prefix zava-storefront` is the truth.
- `docs-generator` has no single natural test. Use its file-scope, TypeScript, test, and comment-only checks from the track guide.
- `dependency-auditor` must not modify `package.json` or lockfiles. It emits a recommendation.
- `npm audit` non-zero is normal on the fixture.

### Golden fallback

Use `docs/golden-examples/` only after the participant has drafted their own skill:

- `docs/golden-examples/test-improver.SKILL.md`
- `docs/golden-examples/docs-generator.SKILL.md`
- `docs/golden-examples/dependency-auditor.SKILL.md`

The fallback is for unblocking, not skipping the design step.

---

## 🔄 Track notes: Framework Modernizer

Guide: `docs/tracks/04-framework-modernizer.md`

This track is a single-skill modernization pipeline. The expected target is a real Next 14 → 15 migration of `zava-storefront/`.

### Done looks like

- The skill has a clear catalog/rubric/pipeline shape for Next 14 → 15.
- Findings cite the upstream migration guide instead of model memory.
- Safe fixes and manual-review items are separated.
- The local run produces a useful migration result against `zava-storefront/`.
- The same skill is packaged and run from CI.

### Common stuck points

- Participants try to build a generic “modernize anything” skill. Keep it scoped to one framework pair.
- The model invents breaking changes. Require citations from the migration guide.
- Regex-only checks may be too weak for some cases. That is fine; the skill should mark those as manual review rather than pretending to fix them.
- People expect the local run and CI run to be identical in timing. Local is fast; CI depends on Actions queue and Copilot availability.

### Golden fallback

Use the reference `framework-modernizer` assets already installed under `.apm/skills/framework-modernizer/` to explain the shape. If a participant's Next 14 → 15 draft stalls, have them compare their design to the reference pipeline and keep only the single-framework-pair structure.

---

## 🏗️ Track notes: Agentic SDLC

Guide: `docs/tracks/agentic-sdlc.md`

This is the advanced track. Participants build three skills in SDLC order and compose them into a loop: a `builder` that implements a change on `zava-storefront` and opens a PR, a `review-panel` that reviews that PR against the participant's own guideline files, and an `sdlc-driver` that runs them in a build → review loop until the PR is clean. Building first means the panel and the loop have a real PR to work on.

### Done looks like

- A `builder` skill reads three team guideline files (security, architecture, documentation), implements a work item (a feature brief, or later a set of review findings), runs the project's deterministic checks (`npm run lint`, `npm test`), and opens/updates a PR on `zava-storefront` only when they pass.
- A `review-panel` skill reviews that PR against the *same* three guideline files and returns one fixed structure: a merge-or-request-changes call, why, and findings ordered by criticality per dimension.
- A deterministic quality floor on the app: ESLint code-quality rules (`complexity`, `max-lines`, `max-lines-per-function`, `max-params`) plus a Vitest coverage threshold. These are the *deterministic gate* — fast, free, certain — that runs before the LLM panel.
- An `sdlc-driver` skill composes both skills as dependencies and loops — run the deterministic checks, then the panel, hand both red checks and findings to the builder, re-check, re-review — until the checks pass and a review returns no actionable findings.
- All three run from the harness as a slash command after `apm install`; ideally they tag a `v0.1.0` release of their own repo (the existing `release.yml` packs the loop), pin it from another repo, and run the review panel in CI via a `gh-aw` workflow gated on a `panel-review` label (label-gated = inference only on demand). Tracing `zava-agent-config` — the org's central catalog — as the at-scale shape, then graduating their loop into it, is the optional last step.

### Common stuck points

- The builder can't open a PR. It works directly against `zava-storefront` — it needs to push a branch and open a PR there. `gh` is already authenticated from setup, so check the auth/remote first. If a participant can't open a PR at all, have the panel point at any public PR instead.
- The deterministic gate is confused with the panel. Linters/tests/coverage are the cheap, certain gate and run *first*; the LLM panel is the judgment gate for architecture/security/doc intent. The lesson is that the driver runs deterministic-green *then* panel-clean — not one or the other.
- The new ESLint rules / coverage threshold light up a lot of existing code. That's expected and useful — it gives the loop real work. Coach people to treat the numbers as an example floor and tune them, not to chase zero findings.
- People expect Genesis to produce the same design for everyone. It won't — designs and file layouts differ per participant. Coach the *shape* (focused pass per dimension, one fixed output, a real loop), not a fixed structure.
- The builder (or panel) ignores the guideline files and works generically. The whole point is that both read the team's own security / architecture / documentation rules — the builder to follow them, the panel to check them.
- The driver re-implements review or fix logic instead of composing the builder and panel via the `apm.yml` dependency edges.
- The loop runs forever or not at all. It must check, review, build, re-check, re-review, and stop when the checks pass and a review returns no actionable findings (with a sane cap), deferring scope-crossing items.
- Evals are a north star, not a workshop step. If someone wants to run them, point out the token cost and that Genesis scaffolds them — the workshop validates by reading the output critically.
- The CI panel doesn't fire (or fires on every push). It's gated on the `panel-review` label: the run only starts when someone applies the label. Unrelated label/PR events should show as a gray *Skipped* — that's the cost gate working, not a failure. They need `gh label create panel-review` and a recompile (`gh aw compile`) after editing `my-workflow.md`. If the run goes red with `Resource not accessible by personal access token`, it's the org secrets — see the secrets table.

### Golden fallback

Use these after the participant has tried their own design:

- `docs/golden-examples/review-panel/`
- `docs/golden-examples/review-driver/` (a worked orchestrator — its name/phases differ from the participant's `sdlc-driver`; use it for the composition + loop shape).

If someone is behind on the build step, let them point the panel at any public PR instead of one their builder opened. If they're behind on the driver, let them wire a golden skill in so they can still learn the composition edge and the build-and-review loop.

---

## 🎁 Optional bonus: PROVISION

Guide: `docs/accelerators/provision.md`

Use PROVISION for early finishers. It is not a fourth track and should not pull support away from participants still trying to ship their main skill.

Good candidates:

- Finished Basics early and wants a more realistic platform workflow.
- Finished Framework Modernizer and wants to see a different domain.
- Finished Agentic SDLC and wants to package a reusable service-provisioning skill.

Keep the instruction simple: “You are done with the main lab. Start PROVISION and call us only if setup breaks.”

---

## 👥 Running a 50-person room

### Staff the spread

For ~50 engineers, aim for:

- 1 lead facilitator.
- 3–5 TAs.
- Pods of 8–10 people.
- At least one TA comfortable with each common harness in the room.

Ask for harness choice at registration if possible. If not, do a quick show of hands at setup.

### Call timeboxes clearly

Use visible checkpoints:

| Time | Call |
|---:|---|
| +15 min | Everyone should have setup done or be with a TA. |
| +25 min | Everyone should have picked a track. |
| +45 min | Everyone should have a Genesis design. |
| +65 min | Everyone should have a first `SKILL.md` draft. |
| +90 min | Basics and Framework Modernizer should be validating locally. |
| +145 min | Agentic SDLC should have the builder opening a PR, the deterministic gate (lint + tests + coverage) set, and the panel reviewing it — or use the golden fallback. Strong finishers release the loop, pin it, and fire the panel in CI on a `panel-review` label. |
| Final hour | Move from local validation to package, release, CI, and consume. |

Do not let a participant spend 45 minutes debugging a personal draft. If they have tried, give them the golden example and keep them in the five-step flow.

### Handle fast finishers

- First: have them help someone in their pod explain their design.
- Then: point them to PROVISION.
- Strongest engineers: steer them to Agentic SDLC if they have enough time left.

### Handle slow finishers

- Keep them on Basics → `test-improver`.
- If their skill does not converge locally, use the golden example.
- If CI auth is broken, stop debugging after 10 minutes and show the reference CI path.
- Make sure they still package and consume a skill, even if it is the reference one.

---

## 🟢 Run live vs. use reference

Be honest about what you can run live.

Run live when:

- Local setup is green.
- `zava-storefront/` is installed.
- The participant has a first skill draft.
- Validation uses local commands such as `npm test`, `npm audit --json`, TypeScript checks, or file diffs.

Use a reference path when:

- Org PATs are missing or not visible.
- GitHub Actions queue is too slow.
- `gh aw` auth fails with `Resource not accessible by personal access token` or `401 no token`.
- More than ~20% of the room is blocked at the same CI step.

Reference is not failure. The learning goal is that the same packaged skill can run locally and in CI. If CI infrastructure is not ready, show the reference run and keep participants moving.

---

## 🧯 Fast troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `/genesis` does not load | `apm install` did not complete or harness did not refresh. | Rerun `apm install`; check `.agents/skills/genesis/`; restart harness if needed. |
| `gh skill` missing | Old `gh` version. | Upgrade `gh`. Do not install a separate extension. |
| `npm test --prefix zava-storefront` fails before skill work | Storefront setup issue or changed fixture. | Rerun `npm install --prefix zava-storefront`; if still broken, use reference path. |
| `npm audit` fails the shell step | Expected vulnerabilities. | Use `npm audit --json` and parse output; append `|| true` in manual commands. |
| CI says `Resource not accessible by personal access token` | `COPILOT_GITHUB_TOKEN` missing or not visible. | Check `gh secret list --org <org>`; org admin sets with `--visibility=all`. |
| CI says `401 no token` | Same token visibility issue. | Same fix. |
| Package install fails in CI | `GH_AW_PLUGINS_TOKEN` missing or lacks Packages read. | Check org secret and token permission. |
| `gh aw` workflow does nothing on label | Label mismatch or workflow trigger mismatch. | Compare the label in the PR/issue to the workflow `if:` condition. |

---

## 📣 Kickoff script

Use this if you need a short opening:

> Today you are building a skill, not just prompting an assistant. The same artifact should run in your IDE and in CI. Every track follows the same five steps: design, build, validate, ship to CI, consume. Pick the track that matches your comfort level. If you get blocked, we will use a golden example so you still finish the full loop.

Then immediately move into setup. Do not over-explain theory before people have a working repo.
