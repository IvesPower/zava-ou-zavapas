# Bonus lab · Provision

> **Optional early-finisher lab.** If you finish your track early, build a skill that helps a developer request a new service through the Zava golden path.

This is not a fourth track. It is a short bonus exercise that applies the same workshop spine to a different problem: turning a repeatable platform process into a focused `SKILL.md`.

---

## What you are building

Build a `provision-golden-path` skill. The skill should guide a developer from:

> "I need a new service"

through a governed IssueOps request that results in a repository with CI, security checks, OIDC-to-Azure, ownership, and a staging URL already wired up.

The skill does not directly create cloud resources from free text. It collects the right inputs, files or drafts the structured request, watches the platform run when available, and explains what was created.

---

## Why this matters

A golden path makes the safe path the easiest path. Instead of every team hand-rolling repo setup, CI, Azure credentials, security scanning, and ownership rules, the platform provides one governed entry point.

Your skill should make that entry point easier to use without hiding the guardrails.

A good provisioning skill should:

- Ask for the service name, owner team, runtime, and required capabilities.
- Refuse vague or unsafe requests instead of guessing.
- Produce a structured IssueOps request the platform bot can validate.
- Explain the guardrails: branch protection, CODEOWNERS, CI, OIDC, dependency checks, and secure baseline.
- Handle both live and walkthrough mode, because org provisioning may not be wired in every workshop environment.

Optional handbook pointer: [Architectural Patterns Rosetta Stone — Workflow and orchestration patterns](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch19-architectural-patterns-rosetta-stone.html).

---

## 1. Design with Genesis

Invoke Genesis before writing the skill.

```text
/genesis I want a provision-golden-path skill. It must:
- Help a developer request a new service through a governed IssueOps front door
- Collect service name, owner team, runtime, Azure target, and capabilities
- Validate that required fields are present before drafting the request
- Explain the platform guardrails that will be applied
- Support live mode when the provisioning bot is available
- Support walkthrough mode when only reference output is available
- Never create arbitrary Azure resources outside the golden path

Draw an ASCII diagram of the proposed skill architecture and explain the design choices.
```

Read the diagram and rationale. Check that Genesis keeps the skill narrow: intake, validation, request drafting or filing, run observation, and final explanation.

Save the design output under `.apm/skills/provision-golden-path/DESIGN.md` after you create the skill folder.

---

## 2. Build the skill

Ask your agent to implement the design:

```text
Use the genesis skill to implement the provision-golden-path skill per the agreed design. Place it at .apm/skills/provision-golden-path/.
```

Review the generated `SKILL.md` before running it. It should have clear boundaries:

- It may draft or file the structured provisioning issue.
- It may read issue comments or reference output to explain progress.
- It may inspect the created repository if one exists.
- It must not bypass the platform bot.
- It must not ask the agent to invent Azure wiring by hand.

If the skill tries to become a general Azure deployment assistant, tighten the scope and regenerate or edit it.

---

## 3. Validate locally

Run the skill against a realistic request:

```text
Use the provision-golden-path skill. I need a TypeScript service called zava-notifications owned by the Storefront team. It needs Postgres and Redis and should deploy to the workshop Azure subscription.
```

Watch for the expected behavior:

1. It asks only for missing required fields.
2. It produces a structured IssueOps request.
3. It explains what the platform will enforce.
4. It clearly labels live steps versus walkthrough steps.
5. It stops at the golden-path boundary.

If live org wiring is available, let the skill file the request and follow the issue comments. If not, point it at reference output and verify that it can explain each platform step without pretending it ran.

---

## 4. Package it

Run the same packaging flow as your track skill:

```bash
apm pack --archive
ls build/
tar tzf build/provision-golden-path-0.1.0.tar.gz
```

Confirm the bundle includes the runtime skill files, not author-time tooling. The important artifact is the `SKILL.md` that another engineer or CI workflow can load.

---

## 5. Optional: run it in CI

Only do this if you have time and the workshop environment supports it.

Use `gh aw` to run the skill from a labeled issue or manual workflow. The workflow should load the packaged skill, process a provisioning request, and post a short status comment. Keep permissions tight: read repository context, write one issue comment, and do not grant broad cloud access to the workflow itself.

The platform bot remains responsible for provisioning. Your skill is the intake and explanation layer.

---

## What you should have learned

- A provisioning skill is useful because it makes a governed platform flow easier to use.
- The skill should collect, validate, and explain. The platform should provision.
- Local validation still matters, even when the final workflow depends on org infrastructure.
- The same five-step workshop spine works for bonus labs: design with Genesis, build, validate, package, and optionally ship to CI.
