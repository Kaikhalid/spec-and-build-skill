---
name: spec-and-build
version: "4.1"
description: >
  End-to-end workflow for planning and implementing any significant codebase change.
  Trigger when the user asks to plan, spec, design, or implement a structural change —
  new features with cross-file impact, refactoring, security hardening, rearchitecting
  a component, migrating a data model, or integrating an external service. Also trigger
  for "write a dev spec", "build a phased plan", "spec and build", "architect this change",
  "spec this out", "plan this feature", or any request combining a structured plan with
  implementation. Runs a for/against/middle-way design argument, produces a phased spec
  with per-phase validation gates, awaits approval, then implements phase-by-phase with
  full test coverage. Use even for requests that sound like simple fixes — if touching
  more than 2 files or involving a gate, permission, state change, or data model, this
  structured approach prevents phantom bugs and incomplete implementations.
---

# spec-and-build — Architectural Spec and Phased Build

**Invocation:** `/spec-and-build <short problem statement>`

A six-stage workflow that separates thinking from building. The core invariant: **never conflate planning and implementation.** If the user says "write a spec" or "develop a plan", deliver the spec and stop. Only build when explicitly told to proceed.

---

## Stage 1 — Deep Read

**Always runs first. Never skip.**

### 1.1 Identify scope

Before forming any opinion, read the system. Run these reads in parallel:

- Grep for all files containing the concept being changed (use relevant keywords)
- Read 40–80 lines of context around every relevant function — not just the matching line
- Find all callers of every function you will touch
- Find all tests that currently cover the area
- Check data models, schemas, or API contracts if state or interfaces are involved

### 1.2 Build a dependency map

Write out (in working notes, not in the spec):

```
Symbol / path → what it does → what calls it → what it affects
```

Do not skip peripheral symbols. The most dangerous bugs come from symbols that seem unrelated — secondary helpers, guard functions, resolver utilities, middleware — that silently affect the behaviour of the mechanism being changed.

### 1.3 Enumerate all affected paths

For any gate, permission check, or data flow being modified: explicitly ask **what are all the ways this mechanism is invoked, bypassed, or depended upon?** List every path. Do not stop at the obvious one.

**Stop condition:** All touch points, callers, and affected paths identified.

---

## Stage 2 — The Argument

**Mandatory for any change that removes an existing mechanism, changes a system boundary, or alters how state, permissions, or data flow are evaluated.**

For purely additive changes (new feature that does not alter existing paths), this stage can be abbreviated to a single paragraph confirming no existing mechanism is being removed or weakened.

### 2.1 Argue FOR the status quo

Write a genuine defence of the current design:

- What it was designed to solve (original intent)
- What it does well (real benefits, not strawmen)
- What would be lost if removed entirely
- Who or what relies on it (callers, users, downstream systems, contracts)

*Goal: produce the strongest possible case for keeping things as they are. If you cannot make a compelling case, note why the existing design is indefensible.*

### 2.2 Argue AGAINST the status quo

Write a rigorous critique:

- The specific failure mode that motivated this review (with evidence: logs, error traces, observed behaviour)
- The root cause (not the symptom) — trace back to the architectural decision, not just the bug or limitation
- The class of problem — is this one instance or a pattern? List all instances of the same class in the codebase
- What the design gets wrong at the conceptual level
- The cost of inaction — what breaks, degrades, or becomes impossible if nothing changes?

*Goal: produce the strongest possible case for removing or replacing the mechanism entirely.*

### 2.3 The Middle Way

The middle way is not a compromise. It is a design that is strictly better than both alternatives:

- It preserves the genuine value of the status quo
- It removes the problem cleanly and completely
- It replaces the mechanism with a better one at the right layer
- It is informed by first principles

**Template:**

```
The middle way preserves [what the status quo did well] by [new mechanism].
It removes [the problem] by [architectural change].
The replacement mechanism is [description] because [principle].

Invariant: [the one thing that must be true before and after the change]
```

**Design principles to apply:**

| Principle | Application |
|-----------|-------------|
| **Purpose limitation** | Each mechanism's scope must match its declared intent. A utility built for one concern must not silently assume responsibility for another. |
| **Explicit over inferred** | Explicit contracts (typed interfaces, injected dependencies, declared configs) are always more robust than inferred or ambient state. |
| **Default safety** | Opt-in changes require explicit action. Defaults must not weaken or bypass existing guarantees through passive use. |
| **Separation of concerns** | Observation ≠ permission. Analytics ≠ authorization. Transport ≠ business logic. Keep layers distinct. |
| **Minimal surface** | Every additional dependency, bypass path, or coupling point is maintenance burden and risk. Remove paths; add explicit contracts. |
| **Reversibility** | Prefer changes that can be rolled back per phase. Migrations, schema changes, and interface removals should have a defined rollback strategy. |

**Stop condition:** Middle way principle stated and invariant defined.

---

## Stage 3 — The Dev Spec

Write the spec as a named Markdown file: `_[descriptive_name].md`

Save to a path the user specifies, or default to a `specs/` or `docs/` directory at the project root if none is given.

### 3.1 Spec structure

```markdown
# [Title] — Architectural Specification
**Version:** 1.0
**Date:** [today]
**Status:** Awaiting Approval
**Supersedes:** [what this replaces, or "nothing"]

## 1. Purpose and Design Principles
[3–5 core principles, each in one sentence]

## 2. Current State: What Exists and What Must Change
### 2.1 Problem Statement
[The failure mode, limitation, or gap — with evidence where available]
### 2.2 Root Cause
[Exact code chain or design decision with arrows: A → B → C → D]
### 2.3 Files with Direct Dependencies
| File | Symbol / Interface | Role | Action |
[one row per symbol that will be touched]
### 2.4 Current Schema / Contract (if data model or API is involved)
[Exact schema, type definition, or interface as it currently exists]

## 3. Target Architecture
### 3.1 [Mechanism / Feature] (post-implementation)
[Exact code or pseudocode of the target design]
### 3.2 [Flow diagram if a new multi-step flow is introduced]
### 3.3 Target Schema / Contract (if data model or API is involved)

## 4. Implementation Phases
[one section per phase — see phase template below]

## 5. Cross-Phase Invariants
[Conditions that must hold at every phase boundary]

## 6. Dependency Map
[Phase → requires → produces]

## 7. What This Spec Does Not Cover
[Explicit scope boundaries — what is deliberately out of scope]
```

### 3.2 Phase section template

Repeat for each phase:

```markdown
### Phase N — [Short title]

**Objective:** [One sentence: what is true at the end of this phase that was not true at the start]

**Scope:** [Exact list of files touched in this phase]

#### N.1 Changes

**[filename — absolute path]**

1. Remove: [exact string, copy-pasted, indentation included]
2. Replace with: [exact replacement, ready to write]
3. [Additional changes in the same file]

**[other filename — absolute path]**

4. [Change]

#### N.2 Validation Gate

All of the following must pass before Phase N+1 begins:

- [ ] [Structural assertion — does this symbol still exist? does this string appear?]
- [ ] [Behavioural assertion — does this function return this value under this input?]
- [ ] [Schema / contract assertion — does this field exist / not exist?]
- [ ] Full test suite passes

**Regression tests for Phase N** (new, in the project's test directory):

- TestPhaseN[ShortName]:
  - test_[assertion_1]
  - test_[assertion_2]
```

### 3.3 Rules for writing phase content

Every change in the spec must include:

1. **The exact old string** — copy-pasted from the file, indentation included. Not a paraphrase. Not a description. The literal string.
2. **The exact new string** — the complete replacement, ready to write.
3. **The file path** — absolute, not relative.
4. **A verification command** — a shell command that confirms the change was applied (grep, jq, curl, language-specific assertion, etc.).
5. **A test that fails before the change and passes after it.**

**Phase ordering rules:**

- Phase 1: pure removal of the problem or obsolete mechanism. No new features. System at Phase 1 end is identical to current system minus the problem.
- Phase 2: neutral infrastructure (schema migrations, new constants, new helper functions, new interfaces).
- Phase 3+: new behaviour, incrementally.
- Last phase: hardening — cleanup of legacy state, documentation update, full audit.

**Phase boundary rule:** No phase begins until the previous phase's validation gate is fully satisfied. A partial gate does not qualify. Fix gate failures in the current phase before advancing.

**Stop condition:** Spec is complete. All phases, gates, and test targets defined.

---

## Stage 4 — Approval Gate

After delivering the spec, **stop**. Do not proceed to code.

Output exactly:

```
Spec complete. [N] phases, [M] validation gates, [K] test targets.

Phase summary:
  Phase 1: [title] — [one-line objective]
  Phase 2: [title] — [one-line objective]
  ...

Awaiting your instruction to proceed. When ready, say:
  "proceed to implement end-to-end"   — runs all phases in sequence
  "proceed to Phase 1 only"           — runs Phase 1 and stops
  "proceed to Phase N only"           — runs Phase N and stops
```

If the user asks questions, answer them. If they request spec changes, update the spec. Do not write code.

---

## Stage 5 — Implementation

**Triggered by:** "proceed to implement" / "implement end-to-end" / "implement Phase N"

### 5.1 One phase at a time

Implement Phase 1. After implementation:

1. Run the Phase 1 validation gate (structural checks + test suite)
2. Report results
3. If any gate item fails: fix it before advancing. Do not proceed with a failing gate.
4. If all gate items pass: proceed to Phase 2 (or stop if "Phase 1 only" was requested)

Repeat until all phases complete.

### 5.2 Implementation discipline

**Read before every write.** Never edit a file you haven't read in this session.

**Prefer surgical edits.** Change exactly what the spec says. Do not refactor adjacent code unless the spec requires it.

**Use patch scripts for complex replacements.** For changes spanning more than 5 lines, or involving string escaping, write a patch script in the project's primary language and execute it:

```python
# /tmp/patch_phaseN_[description].py  (Python example)
with open("/absolute/path/to/file.py", "r") as f:
    src = f.read()
assert "OLD_STRING" in src, "ABORT: target string not found — check if already patched"
src = src.replace("OLD_STRING", "NEW_STRING", 1)
with open("/absolute/path/to/file.py", "w") as f:
    f.write(src)
print("✅ Applied: [description of change]")
```

> For non-Python projects, use the equivalent: a Node.js script, a shell `sed` with `--in-place`, a Go patch utility, etc. The pattern — read, assert presence, replace, write, confirm — is language-agnostic.

The `assert` (or equivalent guard) is mandatory. If the target string is not found, the script must abort rather than silently writing a broken file.

**Verify every change immediately** after applying:

```bash
grep -n "new_symbol_name" /path/to/file   # Must return at least one match
grep -n "old_symbol_name" /path/to/file   # Must return zero matches
```

**Only restart services** when necessary to make new functions, processes, or capabilities available.

### 5.3 Test file structure

Create test files in the project's established test directory and framework. Use one test class or describe block per phase. Language-agnostic template:

```
[tests/test_spec_name] or [__tests__/spec_name.test.ts] or equivalent

Phase 1 test group:
  - [old_thing]_is_removed          — structural absence
  - [new_thing]_is_present          — structural presence
  - [behaviour]_on_[condition]      — behavioural assertion
  - [schema/contract]_is_valid      — data model assertion

Phase 2 test group:
  ...
```

**Test naming conventions (language-agnostic):**

- `[old_thing]_removed` / `removes_[old_thing]` — structural absence
- `[new_thing]_defined` / `defines_[new_thing]` — structural presence
- `[behaviour]_on_[condition]` — behavioural
- `[field]_present_in_schema` — data model / contract
- `gate_has_exactly_[N]_conditions` — invariant count

### 5.4 Final validation

After all phases, run the full test suite and any relevant integration checks. Report:

```
X/Y tests passed.
```

If any fail, fix before declaring done.

---

## Stage 6 — Completion Report

When all phases pass, output:

```
Implementation complete.

Phases delivered:
  ✅ Phase 1: [title] — [gate: N/N items passed]
  ✅ Phase 2: [title] — [gate: N/N items passed]
  ...

Test results: [total] tests, [passed] passed, [failed] failed.

Files changed:
  [file path] — [one-line description of what changed]

What was removed:
  [mechanisms / symbols / fields deleted]

What was added:
  [new mechanisms / symbols / fields introduced]

Invariants confirmed:
  [cross-phase invariants that were verified]
```

---

## Working Principles (always active)

### Delegation discipline

When a sub-task must be delegated, the brief must include:

- The exact file path and line range
- The exact old string and new string
- What the change proves (why it matters)
- The exact command to run to confirm

Never write: "based on your findings, fix the bug." Always write: "in `/absolute/path/to/file` near line N, replace `OLD` with `NEW`, then run `[verification command]` to confirm."

---

## Quick Reference

| Stage | Trigger | Output | Stop condition |
|-------|---------|--------|----------------|
| 1. Read | Skill invoked | Dependency map (internal notes) | All touch points and affected paths identified |
| 2. Argue | Always runs (abbreviated for pure additions) | For / Against / Middle Way | Middle way principle and invariant stated |
| 3. Spec | Always runs | Named `.md` file saved to project | Spec complete, awaiting approval |
| 4. Approval | Spec delivered | Approval prompt | User says "proceed" |
| 5. Implement | "proceed to implement" / "proceed to Phase N" | Phase-by-phase code changes | All phase gates pass |
| 6. Report | All phases complete | Completion summary | All tests green |

---

## Example

**Invocation:**

```
/spec-and-build Add rate limiting to the public API — unauthenticated endpoints are being abused
```

**Expected output sequence:**

1. Deep read of all route definitions, middleware, and existing auth/guard mechanisms
2. For: current open design allows fast iteration and avoids latency overhead for internal consumers
3. Against: no rate limiting creates denial-of-service risk and allows scraping; cost and reliability impact documented
4. Middle way: token-bucket rate limiter at the edge middleware layer, applied per IP for unauthenticated routes, with bypass header for internal trusted consumers using an injected API key — not an ambient IP allowlist
5. Spec: `_api_rate_limiting.md` — 4 phases, 4 validation gates, 12 test targets
6. "Awaiting your instruction to proceed."
7. [On "proceed to Phase 1 only"] Phase 1 implementation + gate verification
8. [On "proceed end-to-end"] Phase 2 → Phase 4 in sequence
9. Completion report: 12/12 tests passed

---

*Skill v4.1 — tightened from v4.0.*
*Changes: removed 'How to use this skill' preamble (superseded by quick reference table); removed Python/TypeScript test boilerplate (Claude Code knows these frameworks); service restart rule updated to necessity-based trigger.*
