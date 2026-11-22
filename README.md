# spec-and-build

A skill for AI coding agents that separates planning from building. You give it a significant codebase change. It reads the system, argues the design, writes a self-contained phased spec, and stops. Code is written only after you approve, and by an operator that checks the spec against the live code first.

Use it for changes that touch more than two files or alter a gate, a permission, persisted state or a data model: new cross-file features, refactors, security hardening, migrations and external-service integrations.

## How it works

The skill has two roles. An **orchestrator** reads the system and writes the spec. An **operator** checks the spec against the live code, builds it and reports. One agent can fill both roles in two phases, or two different models can.

1. **North Star.** State what the change must achieve, for whom, and how anyone can tell. Every later decision traces back to it.
2. **Read.** Find the touch points, callers and tests. Capture what the operator cannot get from the code. Do not try out the plan.
3. **The argument.** Make the strongest case for the current design, the strongest case against it, then state a middle way and the one invariant that must hold before and after.
4. **The spec.** Write a self-contained phased spec. It carries its own Operator Charter, so it works without this skill. Each phase has an Outcome, acceptance checks, a test contract and a rollback.
5. **Approval stop.** Every run stops here. Building starts only on the user's explicit go-ahead, and the spec's Status line records it.
6. **Implementation.** The operator verifies the spec against the code first, then builds one phase at a time. A three-question test decides when a deviation is safe to make without asking. Only five listed Blockers stop the work.

The spec is a guidepost, not hard truth. Locked items (the North Star, Outcomes, invariants, locked decisions and the no-go list) never change. Claims about the code give way to the live code. Suggestions can be changed freely.

## Two versions

| Path | For | Notes |
|---|---|---|
| `SKILL.md` | Claude | v5.0. Invoked as `/spec-and-build <problem statement>`. |
| `codex-plugin/` | Codex | A Codex plugin: manifest in `.codex-plugin/plugin.json`, skill in `skills/spec-and-build/`. Still at v4.1. It follows the older six-stage flow (deep read, argument, dev spec, approval gate, implementation, completion report) and has not been updated to v5.0. |

## Install

**Claude.** Copy the file into a skill folder:

```bash
mkdir -p ~/.claude/skills/spec-and-build
cp SKILL.md ~/.claude/skills/spec-and-build/SKILL.md
```

**Codex.** Copy the skill folder from the plugin:

```bash
cp -R codex-plugin/skills/spec-and-build ~/.codex/skills/
```

I have not tested the Codex copy install from a clean machine. To install it as a full plugin, add `codex-plugin/` to a Codex plugin marketplace. This repo does not include a marketplace file.

## Licence

MIT. See [LICENSE](LICENSE).
