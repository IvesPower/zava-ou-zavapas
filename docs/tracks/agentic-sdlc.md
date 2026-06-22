# Track · Agentic SDLC — compose skills into a build → review loop

> **You've built one skill. Now you'll build three and wire them into a loop.** First a **builder** that implements a change on `zava-storefront` and opens a PR. Then a **review panel** that reviews *that* PR against your team's own guidelines. Then an **`sdlc-driver`** that runs them in a loop — build, review, build again — until the PR is clean. That loop is a miniature SDLC, and by the end you'll see how to keep adding phases (planning, testing, …) to it, one skill at a time.

⏱️ **~145 min**

> 🧭 **Prerequisite.** This track assumes you can already get a single skill from `/genesis` to a working `SKILL.md`. If you haven't done that yet, skim [Track 1 · `test-improver`](01-test-improver.md) first — it's the short version of the move you'll repeat here.

---

## 📚 Theory anchor

- **Live:** [Multi-Agent Orchestration](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch17-multi-agent-orchestration.html) — when one prompt has to hold too much, you split the work across focused skills and have one of them drive the others. That's the loop.
- **Live:** [The Reference Architecture, Earned](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch22-the-reference-architecture-earned.html) — a skill that depends on another skill, pinned by a lockfile. That's the mechanic that turns three skills into one composed unit.

**Local fallback (3 sentences):** A builder that *acts* and a reviewer that *checks* are two halves of a loop; an orchestrator can run build → review → build until the work is done. Each skill is one SDLC phase; the orchestrator composes them by declaring dependencies in `apm.yml`, so `apm install` deploys the whole graph. Add more phase-skills to the same orchestrator and the loop grows from "fix a PR" toward a full agentic SDLC.

---

## 🎯 What you'll build

Three skills, composed into a loop. You'll build them in the order an SDLC actually runs:

1. **`builder`** — given a work item (a feature brief, or later a set of review findings), implements it as commits and opens a PR on `zava-storefront`. It reads three guideline files your team wrote (security, architecture, documentation) so the code it writes already follows your standards. **This is what produces the PR the rest of the track works on.**
2. **`review-panel`** — reviews that PR against the **same** three guideline files and returns one structured answer: a merge-or-request-changes call, why, and findings ordered by criticality per dimension.
3. **`sdlc-driver`** — composes the other two and runs the loop: build the fixes → review again → build again → until the review comes back clean.

Then you'll package all three into a plugin and publish it to a marketplace, so a teammate installs the whole loop with one line.

The lesson isn't "agents are cool." It's the **composition mechanic** — one skill declaring a dependency on another, pinned and deployed like any package — plus the realization that **each SDLC phase is just another skill you snap into the loop.**

> ℹ️ **Everything below is an example, not a spec.** Genesis designs a different skill for every person who prompts it. Your file names, folder layouts, and exact steps will differ from what's shown here — and that's correct. Use these examples to recognize a *good shape*, not to copy a *required structure*.

---

# Part 1 · The builder (35 min)

## 🔍 Discover the problem (5 min)

You want to ship a change to `zava-storefront`. You *could* hand the whole thing to an agent ad hoc — "add this feature" — and copy-paste the result. But you do this all day, and you want it to come out the same way every time: a scoped change, committed on a branch, opened as a clean PR, and — crucially — written to **your team's standards**, not the model's generic defaults.

That's a skill, not a one-off prompt. The builder is the **BUILD phase** of the loop you're about to assemble — the half that *acts*. And the standards it builds to are the same ones your reviewer will later check against, so it pays to write them down once, now.

---

## 🧠 Design with Genesis (10 min)

Prompt Genesis the way you'd describe this to a colleague — plainly:

```
/genesis I want a builder skill. Given a work item — either a short feature
brief or a set of review findings to address — it implements the change as
commits on a new branch and opens (or updates) a pull request on
zava-storefront. It must read my team's guideline files (security, architecture,
documentation) so the code it writes already follows our standards. It should
stay within the work item's scope; anything larger it notes in the PR
description for a human instead of building it. Before it opens or updates the
PR it runs the project's checks (npm run lint and npm test) and only opens the
PR when they pass — if a check is red it fixes that first. Please draw an
architecture diagram of the proposed design in ASCII art.
```

Read what Genesis proposes and check two things: it **reads your guideline files** before writing code (so its output conforms), and it **stays in scope**, flagging anything bigger rather than sprawling. As always, your design will differ from your neighbor's — iterate until it holds up.

---

## 🛠️ Build (20 min)

### Step 1 — Write your three guideline files

This is the part that makes your skills *yours*. Capture what your team checks for — three short markdown files, a handful of rules each is plenty:

- a **security** guideline (e.g. "never read request input without validation", "no secrets in code"),
- an **architecture** guideline (e.g. "new external dependencies need an ADR", "no cross-layer imports"),
- a **documentation** guideline (e.g. "public functions need a doc comment", "user-facing changes update the README").

Write them in your own words — there's no required format. **Both** the builder and the panel will read these, so put them somewhere shared (e.g. a `guidelines/` folder you point both skills at).

### Step 2 — Let Genesis implement, then run it

> Now use the genesis skill to implement the builder per our agreed design.

When it's done, install the workspace deps and run the builder from its slash command with a small feature brief:

```bash
apm install
```

```
/builder add a promo-code endpoint to zava-storefront that applies a percentage
discount to a cart, then open a PR
```

The brief is just an **example** — build anything small with a bit of review surface (it reads input, adds logic, is user-facing). Watch the builder read your guidelines, run the project's `lint` and `test` checks, make the change, push a branch, and open a PR on `zava-storefront` only once those checks are green. **Note the PR number — you'll review it in Part 2 and drive it to clean in Part 3.**

📁 Want a worked reference? [`docs/golden-examples/review-driver/`](../golden-examples/review-driver/) and [`review-panel/`](../golden-examples/review-panel/) show the *shape* of a composed skill — names and phases differ from yours on purpose.

---

# Part 2 · The review panel (40 min)

## 🔍 Discover the problem (10 min)

You've got a real PR now — the one your builder just opened. Try the naive move first. Open a chat and type **"review this PR."** Two things go wrong, and they get worse as the PR gets bigger:

- **Big PRs blow the context.** A large diff is a lot to hold at once. The further the agent reads, the more the early files fall out of focus — so the review near the end is thinner than at the start. Quality drops exactly where you can least afford it.
- **A generic review misses what *your team* cares about.** "Review this PR" reviews it against the model's generic idea of good code. It doesn't know your team bans a particular auth pattern, or requires an ADR for a new dependency, or wants a doc comment on every public function. Those are the findings you actually wanted — and they're the ones a generic pass skips, because nobody told it to look.

You wrote those standards down in Part 1. The fix is to **direct** the review with them: feed it your guidelines, split the review so each pass only holds one dimension, and make it answer in a fixed structure every time so the output is something a human — or another skill — can act on.

That's the panel.

---

## 🧠 Design with Genesis (10 min)

Prompt Genesis plainly:

```
/genesis I want a pr review skill that will review PRs based on specific
security, architecture and documentation guidelines that my team and I have
captured in three different files. The review output should always follow the
same structure (advise with merge or request changes, then why the decision,
then the key findings ordered by criticality, per dimension). PRs can be very
large. Please draw an architecture diagram of the proposed design in ASCII art.
```

Genesis will propose a shape — likely a focused pass per dimension plus a step that assembles the final structured answer, with an ASCII diagram.

**Don't expect the same design we'd draw here.** Genesis is non-deterministic — it'll propose something a little different for everyone, and that's fine. Your job is to read what it gives you and ask whether it solves the two problems from *Discover*:

- Does each dimension get its **own focused pass**, so a large PR doesn't blow one mega-review?
- Does it read your **three guideline files** (the same ones the builder uses) — not generic best practice?
- Is the output a **single, fixed structure** every time (merge-or-request-changes → why → findings by criticality, per dimension)?

If something's off — it inlined everything, forgot the guideline files, or the output drifts — say so and iterate. That conversation **is** the design step.

---

## 🛠️ Build (12 min)

> Now use the genesis skill to implement the review-panel per our agreed design. It should read the same three guideline files the builder uses and always produce the fixed output structure we discussed.

**What's in the folder depends on your design** — one person gets a `SKILL.md` plus a `personas/` folder, the next a single `SKILL.md` with three inline passes. Both can be correct. Here's **one example** layout — yours will differ:

```
.apm/skills/review-panel/              # an example layout, not a required one
├── SKILL.md
├── guidelines/                        # the same three files the builder reads
│   ├── security.md
│   ├── architecture.md
│   └── documentation.md
└── assets/
    └── review-output-template.md
```

Read your `SKILL.md` and confirm it does what the *design* promised:

- [ ] It **reads your three guideline files** — the security pass uses the security guideline, etc.
- [ ] Each dimension is a **focused pass**, so a large PR isn't reviewed in one overloaded read.
- [ ] The output is **one fixed structure**: a merge-or-request-changes call, the reasoning, then findings **ordered by criticality, per dimension**.

📁 Stuck? [`docs/golden-examples/review-panel/`](../golden-examples/review-panel/) is **one** fully-built panel — read it for the shape, not to copy.

---

## ✅ Validate (8 min)

Run your panel on the PR your builder opened in Part 1:

```
/review-panel review PR #<N> on zava-storefront
```

Read the output critically: is the merge-or-request-changes call there? Are findings ordered by criticality, per dimension? Did it catch something *because* it was in your guidelines that a generic review would've skipped? If not, the fix is usually in the guideline files or the persona prompts — iterate. (No PR handy? Point it at any public PR instead.)

> 🌟 **North star — where evals take this.** Reading the output once tells you it works *today*. To trust it in production you'd add **evals**, and for this skill they'd prove two specific things: (1) **format conformance** — every run produces exactly your fixed structure, no drift; and (2) **finding quality against fixtures** — you assemble a set of fixture PRs where you already know what *should* be flagged, then measure whether the panel catches them and ranks them sensibly. Genesis can scaffold exactly that (it has an EVALS PLAN built in — *"use the genesis skill to add evals for this skill"*). We won't run a full eval suite in the workshop — it costs real tokens and time — but that's the direction this goes once it's more than a demo. Evals are how a skill graduates from "looked good once" to "provably still good after every change."

---

# Part 3 · The SDLC loop (40 min)

## 🔍 Discover the problem (5 min)

You've now done both halves by hand: the builder opened a PR (Part 1), the panel reviewed it (Part 2). The obvious next step is to *act on* the review — hand its findings back to the builder, which fixes them and updates the PR. But those fixes might surface *new* findings, so you'd want to review again… and again. Nobody runs that loop by hand. The PR lands with the second-order findings still in it.

The `sdlc-driver` closes the gap, and *how* it does it is the lesson:

- It does **not** reimplement the builder or the panel. It **composes** them — both are dependencies, invoked as self-contained skills.
- It **loops.** Build the fixes → review again → build again → repeat, stopping only when a review returns nothing left to act on.
- It's the smallest possible **SDLC**: two phases (build, review) wired into a cycle. Once you see it, you'll see how to add more.

And not every check needs the panel. Linters, type-checks, and tests give the **same answer every run, for free** — that's your *deterministic gate*. The LLM panel is the *judgment gate*: architecture intent, security reasoning, doc quality — the things a linter can't see. A smart loop runs the deterministic gate **first**: if the code doesn't lint or a test is red, fix it before spending a single token on review. The driver gates on both — deterministic-green, then panel-clean. Those are the same checks a human PR has to pass, so your loop just makes the agent clear the same bar.

---

## 🧠 Design with Genesis (10 min)

```
/genesis I want an sdlc-driver skill that orchestrates a build-and-review loop
on a PR. It should:
- compose two existing skills as dependencies (don't reimplement them):
  my builder (the BUILD phase) and my review-panel (the REVIEW phase)
- each pass, run the project's deterministic checks first (npm run lint and
  npm test with coverage); only invoke the review-panel once they pass, and
  treat a red check as findings to hand straight to the builder
- loop: run the deterministic checks and review-panel, hand any findings to the
  builder to implement, then check and review again — build, review, build —
  until the checks pass and a review comes back with no actionable findings left
- stop on its own with a sane cap so it can't loop forever, and leave
  scope-crossing items for a human
- report a short summary at the end (what changed, what was deferred, how many
  loop passes)
Declare both dependencies in apm.yml so apm install deploys all three together.
Please draw an architecture diagram in ASCII art.
```

Check the design for three things: both skills show up as **dependency edges** (not copied-in logic), the loop runs the **deterministic gate before the panel**, and there's a **real stop condition** — repeat until the checks pass and the review is clean, with a cap. Iterate with Genesis until the loop and the composition are right.

---

## 🛠️ Build (25 min)

### Step 1 — Set the quality floor (the deterministic gate)

`zava-storefront` already lints (`next lint`) and tests (`vitest run`) — but a passing lint says nothing about function length, and a passing test suite says nothing about how *much* is tested. Give those gates teeth so they catch the things a reviewer shouldn't waste time on.

Add code-quality caps to the ESLint config — these mirror what `ruff` enforces in Python (module and function length, branch complexity):

```jsonc
// eslint config — rules block. Numbers are an example floor; tune for your team.
"rules": {
  "complexity": ["error", 10],
  "max-lines": ["error", 300],
  "max-lines-per-function": ["error", 60],
  "max-params": ["error", 4]
}
```

And a coverage floor, so "tests pass" also means "enough is tested" — Vitest:

```bash
npm i -D @vitest/coverage-v8
```

```ts
// vitest.config.ts — test block
coverage: {
  provider: 'v8',
  thresholds: { lines: 70, functions: 70, branches: 70, statements: 70 },
}
```

Now `npm run lint` and `npm run test -- --coverage` are **deterministic gates**: same input, same verdict, every run, no tokens. The driver will run these *before* the panel — and because they run on real code, they'll usually surface honest work for the loop to chew on.

### Step 2 — Let Genesis implement

> Now use the genesis skill to implement the sdlc-driver per our agreed design.

The exact files are yours; here's one possible shape:

```
.apm/skills/sdlc-driver/               # an example layout, not a required one
├── SKILL.md
└── apm.yml
```

### Step 3 — The `apm.yml` dependency edges (this is the lesson)

The thing that *must* be there is the composition — **two** edges, one per phase:

```yaml
name: sdlc-driver
version: 0.1.0
description: >-
  Runs a build → review loop on a PR: reviews with the panel, implements the
  findings with the builder, and repeats until the review is clean.
type: skill

dependencies:
  apm:
    - ../builder
    - ../review-panel
```

What those edges buy you:

- Each path is a **dependency the resolver follows at install time.** Run `apm install` on the driver and both skills deploy alongside it — guidelines, assets, and all.
- The **lockfile** (`apm.lock.yaml`) pins the exact versions this driver was tested with, so consumers get a reproducible graph.
- In a published marketplace, those local paths become versioned references (e.g. `review-panel@^0.1.0`).

It's the same mechanism that makes `npm install` pull a package plus its dependencies at pinned versions — you're building the edges by hand.

### Step 4 — Confirm the loop, then run it

Read your `SKILL.md` and confirm it **composes both skills** (invokes `../builder` and `../review-panel`, doesn't re-implement them), runs the **deterministic gate before the panel** (lint + tests + coverage first, a red check handed to the builder), and **loops to a stop condition** (check → review → build → repeat until the checks pass and no actionable findings remain, with a cap, deferring scope-crossing items).

Then drive the same PR from Part 1:

```bash
apm install
```

```
/sdlc-driver run on PR #<N> on zava-storefront
```

Watch it run the checks, review the PR, hand both the red checks and the panel findings to the builder, update the PR, and **go around again** — looping until the checks pass *and* the panel comes back clean. That repeat is the whole point: confirm it actually re-runs and stops on its own.

📁 Stuck? [`docs/golden-examples/review-driver/`](../golden-examples/review-driver/) is a worked orchestrator showing the `apm.yml` edge and a loop — its name and exact phases differ from your `sdlc-driver`.

---

## 🔁 Keep expanding the loop

You just built a two-phase SDLC — BUILD and REVIEW — out of two skills and an orchestrator. Nothing about the driver is specific to those two phases. **Every SDLC phase is just another skill you snap into the same loop:**

- a **planning** skill that turns an issue into a scoped brief, run *before* the builder,
- a **testing** skill that writes and runs tests on what the builder produced, run *between* build and review,
- a **release** skill that cuts a version once the review is clean.

Each is a skill Genesis can design with you; each becomes one more dependency edge in the `sdlc-driver`'s `apm.yml` and one more step in its loop. That's the same SDLC ribbon the Zava platform organizes its kits around — a skill (or plugin) per phase, composed into one agentic SDLC. You've built the smallest version of it; the rest is more edges.

---

## 📦 Package and ship — release it, pin it, run it, catalog it (30 min)

You've got three composed skills running in your harness. The last move makes them *other people's* skills — and there are two repos in play, not one. **Your repo** is where you authored and ran the skills. The **catalog** is a separate repo where skills graduate into versioned plugins that any team can pin. Keep them straight: you never move your working skills out of your repo to ship them. You release from where they live, and the catalog is its own thing.

### Step 1 — Release your loop from your own repo

Your three skills already sit at `.apm/skills/`, where `apm install` deploys them to your harness — leave them there. This repo already ships a release pipeline at [`.github/workflows/release.yml`](../../.github/workflows/release.yml): on a `v*.*.*` tag it checks the tag matches your `apm.yml` version, validates each skill against the agentskills.io spec, packs them with `apm pack`, and publishes a GitHub Release with the tarball + a `sha256` checksum.

Set the version in `apm.yml`, then cut the tag:

```bash
# apm.yml → version: 0.1.0
git tag v0.1.0
git push origin v0.1.0
```

Watch the run under the repo's **Actions** tab. When it's green, the **Releases** page has your packed loop — and nothing moved: your harness still runs all three skills locally.

### Step 2 — Pin it like a consumer

Now another repo — a teammate's, or a second repo of your own — consumes your loop by pinning that exact tag. (`apm install` resolves the pin from the **git tag**: it fetches the repo at `v0.1.0`, reads its `apm.yml`, and deploys the skills — the Release tarball from Step 1 is a separate download channel, not what the pin uses.) The release notes print the line for you:

```yaml
# their apm.yml
dependencies:
  apm:
    - <your-org>/<your-repo>#v0.1.0
```

`apm install`, and they get the **whole composed graph** — all three skills, at the version you tested. That `#v0.1.0` is the contract: pin a version, get exactly what shipped, never a moving target. (It's the same syntax this workshop's `apm.yml` already uses to pin its kits — go look.)

That one line is how your loop travels: no copy-paste, no vendored skill files — a version-pinned dependency a teammate can adopt and update on their terms. You've now released it and proved another repo can consume it. Next, make CI one of those consumers.

### Step 3 — Run your review panel in CI (label-gated)

Pinning proved the loop is portable. Now make your **CI** one of those consumers — so the review panel runs on a real PR without anyone opening their harness. And run it the way you would for real: **only when someone asks**, not on every push.

The template ships a starter [`gh-aw`](https://github.github.com/gh-aw/) workflow at [`.github/workflows/my-workflow.md`](../../.github/workflows/my-workflow.md) — markdown with a YAML frontmatter (triggers + permissions) and a natural-language prompt. Replace its contents with this. Two lines carry the weight: `packages:` pins the release you just cut, and `if:` gates the run on a single label.

```markdown
---
on:
  pull_request:
    types: [labeled]
  workflow_dispatch:
    inputs:
      pr_number:
        description: "PR number to review"
        required: true
  roles: [admin, maintainer, write]

# Cost gate: run ONLY when a maintainer applies the `panel-review` label.
# gh-aw propagates this `if:` to the activation jobs, so every other label
# event is a gray "Skipped" — no agent start, no inference billed.
if: |
  (github.event_name == 'pull_request' && github.event.label.name == 'panel-review')
  || github.event_name == 'workflow_dispatch'

permissions:
  contents: read
  pull-requests: read
  issues: read

network: defaults

engine:
  id: copilot

# Install YOUR released review-panel at run time. shared/apm.md turns this
# `packages:` list into an `apm install` step inside the compiled workflow.
imports:
  - uses: shared/apm.md
    with:
      target: copilot
      packages:
        - <your-org>/<your-repo>#v0.1.0

safe-outputs:
  add-comment:
    max: 1
  # Drop the label after the run, so re-applying it re-runs the panel.
  remove-labels:
    allowed: [panel-review]

timeout-minutes: 20
---

# Run the review panel

Run the **`review-panel`** skill against PR
**#${{ github.event.pull_request.number || inputs.pr_number }}** in `${{ github.repository }}`.
Read the diff with `gh pr diff` — never check out the PR head. Follow the skill's `SKILL.md`
exactly: review against the guideline files and post the panel's single structured verdict
(advise merge / request changes, the reasons, findings by criticality) as one comment.
Do not modify files, merge, or label the PR.
```

Then create the label, compile, and push:

```bash
gh label create panel-review --color B0E0FF --description "Run the review panel on this PR"
gh aw compile        # → .github/workflows/my-workflow.lock.yml
git add .github/workflows/ && git commit -m "ci: review panel on panel-review label"
git push
```

Open a small PR in your repo, label it `panel-review`, and watch the panel post its verdict — the *same* `review-panel` SKILL.md you ran in your harness, now running unchanged in CI. Apply the label again to re-run; push to the PR *without* re-labeling and nothing fires.

> 💡 **This is the cost lever.** An agent run costs inference. Triggering on *every* PR — or on every push via `synchronize` — bills a model call you never asked for. Gating on a label means the panel runs when a human decides it's worth it, and unrelated label events show as a gray *Skipped*, not a charged run. The production [`microsoft/apm` panel](https://github.com/microsoft/apm/blob/main/.github/workflows/pr-review-panel.md) gates the same way.

> 💡 **CI auth.** `engine: copilot` needs the org secret `COPILOT_GITHUB_TOKEN`, and pulling your pinned package needs `GH_AW_PLUGINS_TOKEN`. If a run fails with `Resource not accessible by personal access token`, an org admin sets them with `--visibility=all` (see [FACILITATOR.md](../FACILITATOR.md)) — you don't need your own PAT.

The duality is the whole point: one `review-panel` SKILL.md, written once, runs **in your harness** while you iterate and **in CI** on a labeled PR — and a teammate pins the same line to run it in theirs. Re-read the README's [§2 payoff](../../README.md#-section-2--the-payoff--your-skill-my-code): **your skill, my code.**

### Step 4 — See the central catalog

One repo per skill doesn't scale to an org. That's what a **catalog** is for — and your org already has one: [`zava-agent-config`](https://github.com/hackathon-brown-eagle-55/zava-agent-config), the same marketplace this workshop's `apm.yml` pins its kits from. Open it and trace the shape:

- **Each plugin is self-contained** — `plugins/<kit>/.apm/skills/<skill>/SKILL.md` plus its own `apm.yml` and `.claude-plugin/plugin.json`. There's one plugin per SDLC phase (provision, ideate, code, review, release, operate). That's the SDLC ribbon — the same one the Zava platform organizes its kits around.
- **The root [`apm.yml`](https://github.com/hackathon-brown-eagle-55/zava-agent-config/blob/main/apm.yml) declares the marketplace** — a `packages:` list where each `source: ./plugins/<kit>` points at a plugin folder, with `build.tagPattern: "v{version}"` tying releases to version tags.
- **Its [`release.yml`](https://github.com/hackathon-brown-eagle-55/zava-agent-config/blob/main/.github/workflows/release.yml) packs *every* plugin on a tag** via `microsoft/apm-action@v1` in `release` mode — the multi-plugin version of the single-skill pipeline you just ran in Step 1.

Same mechanics you used in Steps 1–2, at org scale: many plugins, one marketplace, all versioned by tag.

### Step 5 — Graduate your loop into the catalog (optional)

When a skill earns its place, it graduates from your repo into the catalog as a plugin:

```bash
apm plugin init plugins/sdlc-loop
```

Place a copy of your three skills under `plugins/sdlc-loop/.apm/skills/`, register it in the catalog's `marketplace:` block (`source: ./plugins/sdlc-loop`), and it rides the same tagged release as every other kit. The point worth keeping: the catalog is a *separate* repo from where you author — you graduate a snapshot in, and your working skills stay home where `apm install` still runs them.

---

## 🎓 What you learned

- **Build to a standard.** The builder reads your guideline files before it writes code, so what lands in the PR already follows your team's rules — fewer findings to fix later.
- **Direct the review.** A generic "review this PR" misses what your team cares about and thins out on big diffs. The same guideline files + focused passes + a fixed output shape fix both.
- **Compose, don't reimplement.** The driver depends on the builder and the panel; it doesn't copy them. `apm.yml` declares the edges, the lockfile pins them, `apm install` deploys the closure.
- **Gate deterministically, then by judgment.** Linters, tests, and coverage are free and certain — run them first and let the agent clear the same bar a human PR does. Save the LLM panel for what they can't see: architecture, security, and doc *intent*.
- **Loop to a stop condition.** Check → review → build → repeat until the checks pass and the review is clean — second-order findings don't get left behind, and scope-crossing items go to a human.
- **Every phase is a skill.** Plan, test, release — each is one more dependency edge and one more loop step. That's how a two-phase loop grows into a full agentic SDLC.
- **Evals are the north star.** For this work they'd lock in output-format conformance and finding quality against fixtures — Genesis scaffolds them; that's where you take it past a demo.
- **Release, then pin.** Tag your repo and the pipeline packs your loop; a teammate pins `#v0.1.0` and gets exactly what you tested — your working skills never leave home. Skills graduate into a central catalog as versioned plugins, one per phase.
- **Make CI a consumer — and gate it.** The same `review-panel` SKILL.md you ran in your harness runs unchanged in a `gh-aw` workflow, pinned to your release. Trigger it on a `panel-review` label, not every PR or every push — agent runs cost inference, so you pay only when a human asks for the review.

---

## 🔗 Going further

- The build → review loop is the smallest agentic SDLC. Add planning, testing, and release skills as more edges on the `sdlc-driver` and you've got the full ribbon — the same shape the Zava platform composes from its phase kits.
- The same focused-passes-plus-one-driver shape fits a **migration with interacting axes** — e.g. a framework bump that also forces a breaking dependency upgrade. The [Framework Modernizer track](04-framework-modernizer.md) builds the single-axis version and points at where a loop takes over.
- [Multi-Agent Orchestration](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch17-multi-agent-orchestration.html) and [The Reference Architecture, Earned](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch22-the-reference-architecture-earned.html) are the chapters behind this track.
