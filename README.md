# spec-and-build

A skill for AI coding agents that separates planning from building. You give it a significant codebase change. It reads the system, argues the design, writes a phased spec, and stops. It writes code only after you approve.

Use it for changes that touch more than two files or alter a gate, a permission, persisted state or a data model: new cross-file features, refactors, security hardening, migrations and external-service integrations.

## The six stages

1. **Deep read.** Find every touch point, caller, test and affected path before forming an opinion.
2. **The argument.** Make the strongest case for the current design, the strongest case against it, then state a middle way and the one invariant that must hold before and after.
3. **The dev spec.** Write a phased Markdown spec. Each change carries the exact old text, the exact new text, a verification command and a test that fails before and passes after.
4. **Approval gate.** Stop. A request for a plan is not permission to change code.
5. **Implementation.** Build one phase at a time. A phase does not start until the previous validation gate passes.
6. **Completion report.** List the phases delivered, test results, files changed, what was removed and added, and the invariants confirmed.

## Two versions

| Path | For | Notes |
|---|---|---|
| `SKILL.md` | Claude | Invoked as `/spec-and-build <problem statement>`. |
| `codex-plugin/` | Codex | A Codex plugin: manifest in `.codex-plugin/plugin.json`, skill in `skills/spec-and-build/`. |

The two share the same six stages. The Codex version is a separate rewrite. It reads `AGENTS.md` first, edits through `apply_patch`, and adds failure-path and rollback sections to the spec.

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
