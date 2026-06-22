# 🧰 Zava Skills Workshop Template

> **Build an AI skill once. Run it unchanged in two places — in your own agent while you code, and in CI on every teammate's PR.** That's what turns a personal prompt into team infrastructure. You'll design a skill with [Genesis](https://github.com/DevExpGbb/genesis), build it locally, validate it against a real app, package it with `apm`, and run it in CI with `gh aw`.

[![Use this template](https://img.shields.io/badge/Use%20this-template-brightgreen?logo=github)](../../generate)
[![apm](https://img.shields.io/badge/apm-required-blue)](https://github.com/microsoft/apm)
[![gh-aw](https://img.shields.io/badge/gh--aw-required-blue)](https://github.com/githubnext/gh-aw)

---

## 🤔 The problem this solves

You've used an AI coding assistant. You've felt the ceiling:

- The same task gives **different output every time**.
- Your team's conventions get **silently ignored**.
- Whatever you build lives in *your* IDE — it doesn't ship, it doesn't compose, and it never runs on someone else's PR.
- Every prompt is a one-off. The second one doesn't reuse the first.

A **skill** is the way out: a packaged, versioned unit an agent loads — narrow scope, defined inputs, a tool boundary. But "write a skill" with no scaffolding just gets you another big markdown file. This workshop has you build one the right way: designed first, scoped tight, packaged for distribution, and automated in CI.

---

## 🎯 What you'll build

One skill, taken all the way:

1. **Designed** with [Genesis](https://github.com/DevExpGbb/genesis) — an architecture pass before you write any `SKILL.md`.
2. **Running locally** in your IDE against [`zava-storefront`](https://github.com/DevExpGbb/zava-storefront) — a real Next.js + Postgres commerce app (cloned alongside this repo, not vendored).
3. **Packaged + released** as a versioned tarball (`v0.1.0`) with `apm`.
4. **Running in CI** on labeled PRs via [`gh aw`](https://github.com/githubnext/gh-aw) — the *same* skill, no rewrite.
5. **Consumed by another repo** that pins your skill in its `apm.yml`.

The whole point is steps 2 and 4 being the **same artifact**. Build it once; run it in your inner loop *and* the team's outer loop.

---

## 🧭 How the workshop runs

Everyone follows the **same five steps** — the only thing that changes is *what* skill you build:

| Step | What you do |
|---|---|
| **Design** | Run `/genesis` on your skill's brief. It returns an architecture diagram + rationale. That's your spec. |
| **Build** | Have your harness generate the `SKILL.md` from the design — you review it, you don't hand-type it. |
| **Validate** | Drive the skill locally against `zava-storefront/` until it does the job. |
| **Ship to CI** | `apm pack` → tag → release → wire it into a `gh aw` workflow that runs on labeled PRs. |
| **Consume** | Pin your released skill from another repo and watch `apm install` pull it in. |

Your **track guide** walks you through all five steps with track-specific content. Pick a track below.

---

## ⚙️ Section 0 · Setup (15 min)

Do this once, before you pick a track.

### Prerequisites

- [`apm`](https://github.com/microsoft/apm):
  - **macOS:** `brew install microsoft/apm/apm`
  - **Linux / WSL:** `curl -fsSL https://raw.githubusercontent.com/microsoft/apm/main/install.sh | bash`
  - **Windows:** see the [APM install docs](https://microsoft.github.io/apm/install) (the bash installer works under WSL).
- [`gh`](https://cli.github.com) (a current version — the `gh skill` command ships inside `gh` now; no separate install).
- [`gh aw`](https://github.com/githubnext/gh-aw): `gh extension install github/gh-aw`.
- An **agent harness**: Copilot CLI / Claude Code / Codex / Cursor / OpenCode.
- **Node.js ≥ 20** (for `zava-storefront/`).

### 1. Use this template

Click the green **"Use this template"** button → **"Create a new repository"**. Pick your org + name, then clone your new repo locally.

### 2. Install dependencies

```bash
# from the root of the repo you generated in step 1

apm install                             # workshop kits + Genesis design assistant

# Clone the target app at the pinned baseline (it's gitignored, so your repo stays clean):
git clone --branch workshop-v1 --depth 1 https://github.com/DevExpGbb/zava-storefront.git zava-storefront
npm install --prefix zava-storefront
```

`apm install` pulls in the four **workshop kits** (`secure-baseline`, `code-kit`, `ideate-kit`, `review-kit`) and **Genesis** — your design assistant (`/genesis` in your harness).

### 3. Verify CLIs and harness

```bash
apm deps list                      # should list the 5 deps (4 kits + genesis)
ls .agents/skills/genesis          # should exist — proves apm install completed

gh auth status                     # logged in to github.com as <you>
gh repo view                       # resolves to your generated repo

# Harness check — type `/genesis` in your agent chat. It should autocomplete or
# respond when you ask "what skills are loaded?". If not, re-run `apm install`.
```

If `/genesis` surfaces in your harness and `apm deps list` shows five deps, you're ready. (Track-specific checks — like `npm test` for the test-improver — live in the track guides.)

---

## 🛤️ Section 1 · Pick your track (5 min)

Same five steps, three levels of ambition. Pick **ONE** based on what you'd actually use at work.

| Track | You build | Time | Best for |
|---|---|---|---|
| 🧰 **Basics** — pick one skill: <br>🧪 [`test-improver`](docs/tracks/01-test-improver.md) · 📖 [`docs-generator`](docs/tracks/02-docs-generator.md) · 🛡️ [`dependency-auditor`](docs/tracks/03-dependency-auditor.md) | One small, sharp skill against `zava-storefront/` | ~90 min | First time building a skill — cleanest loop |
| 🔄 [**Framework Modernizer**](docs/tracks/04-framework-modernizer.md) | One modernization skill, run against a real Next 14 → 15 migration of `zava-storefront/` | ~90 min | You're facing an actual major-version upgrade |
| 🏗️ [**Agentic SDLC**](docs/tracks/agentic-sdlc.md) | Three skills composed into a build → review loop — a builder, a review panel, and a driver that runs a deterministic gate (lint + tests + coverage) then the panel until the PR is clean; then release the loop, pin it, and run it in CI on a labeled PR | ~145 min | You want the full multi-agent picture |

> 💡 **Not sure?** Start with **Basics → `test-improver`**. `npm test` is the oracle, so the validation loop is unambiguous.

Every track ends at the **same payoff**: your skill running in CI, installable by any other repo. The Agentic SDLC track is the deep end of the same pool — not a different destination.

> 🎁 **Finished early?** The optional [**PROVISION**](docs/accelerators/provision.md) lab builds a golden-path service-provisioning skill. It's a bonus, not a fourth track.

---

## 🌐 Section 2 · The payoff — your skill, my code

After you ship to CI (Step 4 of your track), the same skill that runs in your IDE *also* runs as a `gh aw` workflow on labeled PRs. Nothing was rewritten between the two — one artifact, two loops.

Then comes the part that makes it team infrastructure. In **any** consumer repo, drop into their `apm.yml`:

```yaml
dependencies:
  apm:
    - <your-org>/<your-repo>#v0.1.0
```

`apm install` in their repo gives them your skill — same `SKILL.md`, same `allowed-tools` boundary, same lifecycle. They didn't copy your prompt; they pinned a version of it.

That's the difference between an ad-hoc prompt and a packaged skill: **distribution, lifecycle, and semantic versioning** — the things you already expect from npm or pip, now for the way your team works with agents.

---

## 📂 Where to look when you're stuck

You don't need to read this repo end to end. When you get stuck, here's where to look:

- **Your track guide** (`docs/tracks/`) — the step-by-step for the track you picked. Start here.
- **`zava-storefront/`** — the real app your skill works on (cloned in §0.2). Open the files your track names.
- **`docs/golden-examples/`** — finished reference skills. Peek **after** you've drafted yours, to compare — not before.
- **`docs/FACILITATOR.md`** — if you're running the workshop for a room, the facilitator notes and per-track preflight live here.
- **`docs/accelerators/provision.md`** — the optional bonus lab.

The plumbing — `apm.yml`, `.github/workflows/release.yml`, `.github/workflows/my-workflow.md`, the `framework-modernizer` reference skill under `.apm/skills/` — is wired for you. The track guides point at each piece when you need it.

---

## 🎓 What you took home

- **Design before code.** Genesis is the architecture pass that stops a skill from becoming another monolith.
- **Skills ship like packages.** Tag → release → consumers pin. Same lifecycle as npm or pip.
- **Inner loop == outer loop.** Whatever runs in your IDE runs the same way in CI. No translation layer.

---

## 🔗 Going further

- *The Agentic SDLC Handbook* — the theory behind this workshop. Start with [The Reference Architecture](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch04-the-reference-architecture.html), then [The PROSE Specification](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch13-the-prose-specification.html) and the [Architectural Patterns Rosetta Stone](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch19-architectural-patterns-rosetta-stone.html). The Agentic SDLC track also draws on [Multi-Agent Orchestration](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch17-multi-agent-orchestration.html) and [The Reference Architecture, Earned](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch22-the-reference-architecture-earned.html). When you're ready to keep going: [What Comes Next](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch27-what-comes-next.html).
- [`microsoft/apm`](https://github.com/microsoft/apm) and the [APM docs](https://microsoft.github.io/apm/) — the package manager you used to install kits and ship your skill.
- [`microsoft/apm-action`](https://github.com/microsoft/apm-action) — wires `apm install` into any workflow (the building block under `shared/apm.md`).
- [`danielmeppiel/genesis`](https://github.com/danielmeppiel/genesis) — use `/genesis` in your harness on real problems, not just workshop ones.
- Issues / PRs on [zava-workshop-kit](https://github.com/DevExpGbb/zava-workshop-kit) — the deployer-facing entry point for running this workshop in your own org.

> **Honest scope.** This is a hands-on learning kit, not a production rollout. It targets GitHub (incl. Enterprise Cloud) + GitHub Copilot, and the CI track uses `gh aw` with `engine: copilot`. A production setup would add engine abstraction, GitHub App auth (vs. PATs), attestation, and prompt audit logging — out of scope here.

---

## 📝 Workshop reference

This template is **Artifact #5** of the Zava agentic SDLC workshop. Org administrators provisioning the workshop should start at [`DevExpGbb/zava-workshop-kit`](https://github.com/DevExpGbb/zava-workshop-kit).
