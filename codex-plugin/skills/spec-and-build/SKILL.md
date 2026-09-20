---
name: spec-and-build
description: End-to-end architecture and phased implementation workflow for significant codebase changes. Use when the user asks Codex to plan, spec, design, architect, or implement a structural change such as a cross-file feature, refactor, security hardening, component redesign, state or permission change, data-model migration, or external-service integration. Also trigger for "write a dev spec", "build a phased plan", "spec and build", "architect this change", "spec this out", and "plan this feature". Use for apparently simple changes when they touch more than two files or alter a gate, permission, persisted state, schema, or system boundary. Do not use for isolated explanations, read-only reviews, or clearly trivial single-file edits that do not benefit from a formal approval gate.
---

# Spec and Build

Separate architectural planning from implementation. Deep-read the real system, argue the design honestly, write an implementation-ready phased specification, stop for approval, and then build phase by phase only after the user authorizes implementation.

Core invariant: never blur a request for a plan into permission to change code. If the user asks only for a spec or plan, deliver the spec and stop.

## Codex operating contract

- Read every applicable `AGENTS.md` and repository instruction before acting.
- Treat the canonical runtime or checkout named by the user as the implementation source of truth.
- Preserve unrelated dirty-worktree changes.
- Use `rg` and `rg --files` for discovery when available.
- Parallelize independent read-only discovery when useful.
- Use `apply_patch` for local file edits. Use the environment's supported safe edit mechanism for remote-only runtimes.
- Use current official documentation when correctness depends on changing APIs, libraries, standards, or product behavior.
- Never broaden implementation authority beyond the user's request.
- Never expose hidden reasoning, secrets, credentials, or unrelated private data in specs or reports.

## Stage 1 — Deep read

Always run this stage first.

### 1.1 Establish scope and authority

Determine:

- The requested outcome and explicit non-goals
- Whether the user requested planning only, implementation, or both
- The canonical repository, runtime, branch, and deployment boundary
- Applicable repository instructions and available skills
- Existing uncommitted changes and ownership boundaries

Do not write code during discovery.

### 1.2 Identify every touch point

Before forming a design:

- Search for every symbol, configuration key, schema field, route, state, and user-visible behavior involved.
- Read 40–80 lines around each relevant function, not only the matching line.
- Find all callers and consumers of every interface likely to change.
- Find existing tests, fixtures, migrations, manifests, docs, and operational scripts covering the area.
- Check persistence, permission, failure, retry, restart, and rollback paths where relevant.

### 1.3 Build a dependency map

Keep concise working notes in this form:

```text
Symbol or path → responsibility → callers → downstream effects
```

Include secondary helpers, guards, caches, middleware, background workers, and external delivery paths. Peripheral dependencies are common sources of phantom regressions.

### 1.4 Enumerate affected paths

For every gate, permission, state transition, or data flow, ask:

- How is it entered?
- How is it bypassed?
- Which fallback paths exist?
- What persists across retries or restarts?
- What clears or finalizes the state?
- Which tests observe API success but not the rendered or user-visible result?

Stop only when the direct and indirect paths are accounted for.

## Stage 2 — The argument

Run the full argument when the change removes a mechanism, changes a system boundary, or alters state, permissions, or data flow. For a purely additive change that weakens nothing, abbreviate this stage to one paragraph explaining why.

### 2.1 Argue for the status quo

Give the strongest real defense:

- Original intent
- Benefits it currently provides
- Callers, users, or contracts that rely on it
- What would be lost if it were removed

Do not use a strawman.

### 2.2 Argue against the status quo

Give the strongest evidence-backed critique:

- The observed failure mode, including logs, traces, tests, or rendered behavior
- The root cause, traced to the architectural decision rather than the surface symptom
- Other instances of the same failure class
- The conceptual mismatch in the current design
- The cost of leaving it unchanged

### 2.3 State the middle way

The middle way must be strictly better than both keeping and deleting the mechanism:

```text
The middle way preserves [genuine value] by [new mechanism].
It removes [failure class] by [architectural change].
The replacement belongs at [layer] because [principle].

Invariant: [condition that must remain true before and after the change]
```

Apply these principles:

| Principle | Application |
|---|---|
| Purpose limitation | Keep each mechanism inside its declared responsibility. |
| Explicit over inferred | Prefer declared contracts and state transitions over ambient assumptions. |
| Default safety | Do not weaken safeguards through passive defaults. |
| Separation of concerns | Keep observation, permission, transport, persistence, and business logic distinct. |
| Minimal surface | Remove unnecessary paths and coupling. |
| Reversibility | Define rollback for migrations, interface changes, and deployments. |

## Stage 3 — Write the development specification

Create a named Markdown file at the user's requested path. If no location is given, use the repository's established specs or docs directory. If neither exists, create `docs/specs/_[descriptive-name]_dev_spec_v1.md`.

Use absolute paths in file-specific implementation instructions.

### 3.1 Required structure

```markdown
# [Title] — Architectural Specification
**Version:** 1.0
**Date:** [today]
**Status:** Awaiting Approval
**Supersedes:** [prior mechanism or "nothing"]

## 1. Purpose and Design Principles

## 2. Current State
### 2.1 Problem Statement
### 2.2 Evidence
### 2.3 Root Cause
### 2.4 Files and Symbols with Direct Dependencies
### 2.5 Current Schema or Contract

## 3. Target Architecture
### 3.1 Target Mechanism
### 3.2 Target Flow
### 3.3 Target Schema or Contract
### 3.4 Failure and Rollback Behavior

## 4. Implementation Phases

## 5. Cross-Phase Invariants

## 6. Dependency Map

## 7. Explicit Non-Goals
```

Omit schema subsections only when no schema or interface contract is involved.

### 3.2 Phase template

For every phase include:

```markdown
### Phase N — [Title]

**Objective:** [What becomes true]

**Scope:** [Exact files and interfaces]

#### N.1 Changes

**[absolute file path]**

1. Current anchor or exact old contract: [literal text or precise symbol]
2. Replace or extend with: [implementation-ready target]
3. Preserve: [behavior that must not regress]

#### N.2 Validation Gate

- [ ] Structural assertion
- [ ] Behavioral assertion
- [ ] Contract or schema assertion
- [ ] Failure-path assertion
- [ ] Relevant test suite passes

**Regression tests**

- `[test name]` — [what it proves]

**Verification commands**

```bash
[repo-native commands]
```

**Rollback**

[Safe reversal procedure]
```

### 3.3 Specification quality rules

Each planned change must include:

1. The exact file and symbol or stable anchor
2. The current behavior or literal old contract
3. The complete target behavior
4. A verification command
5. A test that fails before the change and passes afterward
6. The invariant the change protects

Phase ordering:

- Phase 1 removes or isolates the failure mechanism without adding unrelated features.
- Phase 2 adds neutral contracts, schema, or infrastructure.
- Phase 3 and later add behavior incrementally.
- The final phase hardens failure paths, removes obsolete compatibility code, updates documentation, and runs the full audit.

Do not begin a later phase while an earlier phase gate is failing.

## Stage 4 — Approval gate

After writing and delivering the spec, stop. Do not edit implementation files, restart services, commit, deploy, or send external messages.

Output:

```text
Spec complete. [N] phases, [M] validation gates, [K] test targets.

Phase summary:
  Phase 1: [title] — [objective]
  Phase 2: [title] — [objective]
  ...

Awaiting your instruction to proceed. When ready, say:
  "proceed to implement end-to-end"
  "proceed to Phase 1 only"
  "proceed to Phase N only"
```

Answer questions and revise the spec when requested. Planning approval is not implementation approval.

## Stage 5 — Implementation

Begin only after explicit authorization such as `proceed to implement`, `implement end-to-end`, or `implement Phase N`.

### 5.1 Re-read before writing

At implementation time:

- Re-read the approved spec and each target file.
- Confirm the anchors still match; if the system drifted materially, stop and update the spec.
- Re-check the worktree and preserve unrelated changes.
- Update a working plan with one in-progress phase at a time.

### 5.2 Implement one phase at a time

For each phase:

1. Apply only the approved changes.
2. Verify structural anchors immediately.
3. Run the phase's targeted tests.
4. Run the phase validation gate.
5. Fix failures before advancing.
6. Report the gate result concisely.

If the user authorized only one phase, stop when that phase gate passes.

### 5.3 Editing discipline

- Prefer surgical patches over broad rewrites.
- Use `apply_patch` for local text edits.
- Do not generate or overwrite files through ad hoc shell redirection.
- Preserve established formatting, APIs, and naming unless the spec changes them.
- Do not refactor adjacent code merely because it could be cleaner.
- Add comments only where they explain a non-obvious invariant.
- Restart services only when required to load or validate the changed runtime.

### 5.4 Testing discipline

Use the repository's established framework and test layout. Cover:

- Structural removal and addition
- Normal behavior
- Boundary conditions
- Failure and fallback behavior
- Persistence and restart behavior when applicable
- User-visible or rendered semantics when presentation matters

Run the broadest maintained suite proportionate to risk. If a known unrelated failure exists, establish the baseline, prove it is unchanged, and report it precisely; never call a failing suite green.

### 5.5 Delegation discipline

Delegate only when the user or applicable instructions permit it. Give each delegated task:

- Exact file and symbol scope
- Current contract and target contract
- What the change proves
- Verification command
- Explicit prohibition on unrelated edits

Treat delegated findings as untrusted until verified against the canonical checkout.

### 5.6 Final validation

After all authorized phases:

- Run targeted regression tests.
- Run the maintained broader suite.
- Run integration, rendering, or live acceptance checks when the user-visible outcome depends on them.
- Inspect the final diff for scope, secrets, generated noise, and accidental deletions.
- Confirm deployment or service health only when deployment was authorized.

Do not declare completion while an in-scope gate is failing.

## Stage 6 — Completion report

Lead with the outcome:

```text
Implementation complete.

Phases delivered:
  Phase 1: [title] — [gate result]
  Phase 2: [title] — [gate result]

Tests:
  [targeted result]
  [maintained-suite result]
  [live or rendered acceptance result]

Files changed:
  [path] — [purpose]

Removed:
  [obsolete mechanisms]

Added:
  [new mechanisms]

Invariants confirmed:
  [verified invariants]

Commit/deployment:
  [only actions explicitly authorized and confirmed]
```

Report blockers or unchanged baseline failures separately. Do not bury them inside a success statement.

## Quick reference

| Stage | Output | Stop condition |
|---|---|---|
| 1. Deep read | Dependency and affected-path map | All touch points identified |
| 2. Argument | For, against, middle way, invariant | Target principle established |
| 3. Dev spec | Named implementation-ready spec | All phases and gates defined |
| 4. Approval | Spec summary and approval prompt | User authorizes implementation |
| 5. Build | Phase-by-phase changes and gates | Authorized phases pass |
| 6. Report | Evidence-backed completion summary | Work genuinely complete |
