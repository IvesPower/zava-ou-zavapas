# Zava Skills Workshop — three-track design (plain voice)

**Status:** approved 2026-06-18. **Supersedes** `2026-06-17-zava-workshop-composition-arc-design.md`
(the "climb / rungs / summit" redesign — rejected for forced vocabulary and for
deleting the track-selection model).

This spec is the contract every writer (human or subagent) follows. Read §1 (voice)
and §8 (file plan) before touching anything.

---

## 1. Audience & voice (non-negotiable)

Write for **working software engineers with no PhD**. The canonical voice reference is
the **original** track guides (`docs/tracks/01-test-improver.md` etc., restored from the
initial commit). Match them exactly:

- Plain, direct, second person: *"You are not fixing the app. You are authoring a Skill that…"*
- Concrete commands, file paths, timeboxes. Show, don't sell.
- **No** forced metaphors or hyperbole. Banned words in the new content: *rung, climb,
  summit, mountain, ascent, brick, factory* (as an arc label), *dark software factory*.
  (A driver that "drives a PR to green" is fine — that's literal.)
- Chapter/handbook links: **minimal**. The README carries chapter links **only** in
  "Going further." A track guide may carry at most one short "why this works" pointer if
  it genuinely helps — not a chapter callout per section.

## 2. The thesis (one idea, stated like an engineer would)

> Build an AI skill once. Run it unchanged in two places — in your own agent while you
> code (inner loop), and in CI on every teammate's PR (outer loop). That's what turns a
> personal prompt into team infrastructure.

This is the original workshop's *"your skill, my code"* insight. It is the centerpiece.
Keep it.

## 3. Structure: one spine, pick-your-track

Everyone does the **same five steps**; the track only changes *what* skill you build and
how ambitious it is:

**Design (Genesis) → Build → Validate → Ship to CI → Consume from another repo.**

### Pick ONE track

| Track | You build | For |
|---|---|---|
| 🧰 **Basics** | One small, sharp skill — choose `test-improver`, `docs-generator`, or `dependency-auditor` | First time building a skill; cleanest validation loop |
| 🔄 **Framework Modernizer** | One modernization skill run against a real, messy migration in `zava-storefront/` | Facing an actual major-version upgrade |
| 🏗️ **Agentic SDLC** | Three skills composed into a build → review loop — a builder, a review panel, and a driver that runs them until the PR is clean | Want the full multi-agent picture |

Every track ends at the **same payoff**: the skill running in CI, then installable by any
other repo. The Agentic SDLC track is the deep end of the same pool — not a separate
destination.

**Bonus (optional):** `PROVISION` — a golden-path service-provisioning lab for anyone who
finishes early. This is the "tracks + accelerators" the user asked to keep. It is **not** a
fourth mandatory track.

## 4. Setup fixes (apply in README + FACILITATOR)

- `gh skill` ships **inside `gh` now** — remove the separate `gh extension install
  github/gh-skill` step and the "preview subcommand" caveats. Assume a current `gh`.
- Move `npm test --prefix zava-storefront` **out** of global setup. It is the
  `test-improver` oracle, not a global requirement — it belongs inside the Basics /
  test-improver track. Global setup verifies CLIs + harness + that `apm install` and the
  storefront clone worked, nothing track-specific.
- **Delete the "Reproducibility caveats" section** entirely.

## 5. README flow (final section order)

1. Title + one-line promise (the thesis, nothing more).
2. The problem (keep original — the AI-assistant ceiling).
3. What you'll build (a skill is a packaged, versioned unit; build one, run it in two places).
4. Setup (global essentials only, with §4 fixes).
5. Pick your track (the §3 table; "stuck? start with Basics").
6. The shared loop (one short paragraph naming the five steps → "your track guide walks you through them").
7. The payoff: local **and** CI — the *"your skill, my code"* section, kept and sharpened.
8. What's in this repo — rewritten as *"where to look when you're stuck"*: a short
   human orientation (target app, track guides, golden examples to peek at after you draft,
   facilitator notes), **not** a file inventory.
9. What you took home + Going further (all chapter links live here, and only here).

## 6. Golden examples (reference skills participants peek at after drafting)

Keep all of:
- `docs/golden-examples/test-improver.SKILL.md`, `docs-generator.SKILL.md`,
  `dependency-auditor.SKILL.md` (+ `dependency-auditor.evals/`) — Basics.
- `docs/golden-examples/review-panel/` and `review-driver/` — the Agentic SDLC track's
  reference artifacts.

## 7. Theory-link policy

Only the verified handbook slugs (renumbered handbook). Quick map for the few links we keep:
`ch04-the-reference-architecture`, `ch13-the-prose-specification`,
`ch19-architectural-patterns-rosetta-stone`, `ch17-multi-agent-orchestration`,
`ch22-the-reference-architecture-earned`, `ch16-deterministic-probabilistic-boundary`,
`ch27-what-comes-next`. **Stale → fix:** `ch12-the-prose-specification` → `ch13-…`;
`ch18-architectural-patterns-rosetta-stone` → `ch19-…`. README chapters only in Going further.

## 8. File plan

### 8.1 Restore from initial commit (95eb750), then only fix stale slugs
- `docs/tracks/01-test-improver.md`, `02-docs-generator.md`, `03-dependency-auditor.md` —
  restore verbatim; fix `ch12→ch13`, `ch18→ch19`. These are the voice reference.
- `.apm/skills/my-skill/SKILL.md` — restore (generic placeholder, applies to all tracks).
- `.github/workflows/my-workflow.md` — restore (generic `run-my-skill` on labeled PRs).

### 8.2 Restore-then-enrich
- `docs/tracks/04-framework-modernizer.md` — restore as base, then **enrich so the in-room
  work is substantive**: participants run the modernizer against the real `zava-storefront/`
  Next 14→15 surface and act on genuine breaking changes, producing an applied migration
  plan — not just "skim a reference and author one catalog row." Keep the single-skill
  pipeline shape. Mine `docs/accelerators/modernize.md` for the "what makes this migration
  genuinely hard" substance, then delete that file. Fix slugs.

### 8.3 New (reshape my redesign content into plain voice)
- `docs/tracks/agentic-sdlc.md` — the advanced track. Fold the content of
  `rung-1-primitive.md` + `rung-2-panel.md` + `summit-factory.md` into ONE coherent guide
  in the original voice: build a skill → compose into a review panel (specialist skills +
  a synthesizer that posts one advisory comment) → add a driver that runs the panel on a
  PR, folds the easy fixes, pushes, and watches CI go green → it runs the same way in CI.
  Reference `docs/golden-examples/review-panel/` and `review-driver/`. No rung/summit words.

### 8.4 Delete
- `docs/tracks/rung-1-primitive.md`, `rung-2-panel.md`, `summit-factory.md`,
  `appendix-alternate-bricks.md` (content folded into agentic-sdlc.md; the 3 easy tracks
  are restored so the appendix is moot).
- `docs/accelerators/modernize.md` (folded into track 04).

### 8.5 Reground (plain voice, light touch)
- `README.md` — full rewrite per §5.
- `docs/FACILITATOR.md` — plain rewrite for the 3-track + bonus model; per-track preflight;
  50-person logistics + golden-example fallback kept; setup §4 fixes applied; curl loop uses
  only §7 slugs.
- `docs/accelerators/provision.md` — keep, reground to plain voice, reframe as optional bonus
  ("if you finish early"), drop "accelerator slot" framing.
- `apm.yml` — keep deps/versions UNCHANGED. Reframe script comments to the restored track
  names. (`regression-track-3` = dependency-auditor; `regression-track-4` =
  framework-modernizer, path `.apm/...` already corrected.)

## 9. PR / canonical logistics

The approved target is **`DevExpGbb/zava-skills-workshop-template`** (canonical), not the
org copy. Constraint: the org repo was template-generated and shares **no git history** with
canonical (`git merge-base` empty), and canonical `main` has diverged (own track edits). So:
1. Close the wrong PR (#1 on `hackathon-brown-eagle-55`).
2. Cut a branch from `DevExpGbb/main`, apply this content there, reconcile with canonical's
   recent track edits file-by-file, open the PR against `DevExpGbb/main`.
Handle after content is written and verified.

## 10. Scope / non-goals

Template DOCS + golden-example scaffolding only. No upstream app changes
(`zava-storefront`, `zava-agent-config`, `genesis`, `awd-cli`), no live LLM end-to-end runs,
no org redeploy. Guides label "run live" vs "reference" honestly.
