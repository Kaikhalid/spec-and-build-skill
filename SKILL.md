---
name: spec-and-build
version: "5.0"
description: >
  End-to-end workflow for planning and implementing any significant codebase change.
  Use it when the user asks to plan, spec, design or implement a structural change: a
  feature that affects several files, a refactor, security hardening, a component
  redesign, a data-model migration, or an external-service integration. Also use it for
  "write a dev spec", "build a phased plan", "spec and build", "architect this change",
  "spec this out", "plan this feature", or any request that combines a structured plan
  with implementation. Use it even for requests that sound like simple fixes if they
  touch more than 2 files or involve a permission check, a state change or a data
  model. An orchestrator writes a self-contained phased spec built around a North Star,
  the run stops for the user's approval, and then an operator builds it.
---

# spec-and-build — Architectural Spec and Phased Build

**Invocation:** `/spec-and-build <short problem statement>`

This workflow separates thinking from building. It has two roles. One agent may fill both roles in two phases, or two different models may fill them.

- **Orchestrator** (Stages 0–4): reads the system, decides the design and writes the spec. It does not build. It does not try out the plan.
- **Operator** (Stage 5): checks the spec against the live code, builds it, verifies it and reports. The operator owns the code.

## Core rules

1. **Every run ends at the approval stop (Stage 4).** Building starts only on the user's explicit go-ahead. "Spec and build" is not a go-ahead.
2. **The North Star governs.** Every decision, deviation and test must trace back to what the change is for.

## Language rules (apply to this skill and to every spec it produces)

1. One instruction per sentence. Keep sentences to 25 words or fewer.
2. Use "must" and "must not" for hard rules. Use "may" for permissions. Do not use hedges such as "usually", "tends to", "about" or "wherever possible" in a rule.
3. Define each term once, and use it with the same meaning everywhere. The spec's terms (Locked items, Claims, Suggestions, Outcome, touch set, Blocker and others) are defined once, in Charter item C1 (section 3.3).
4. Use plain verbs. Do not put metaphors inside rules.
5. Use numbers instead of "small" or "large".
6. Use British spelling.

---

## Stage 0 — North Star

Before you read code, write a first draft answer to this question: **"After this is implemented, what should it have achieved for the system or feature, for whom, and how can anyone tell?"** Revise the draft after Stage 1.

A spec that says only *what* to change lets an operator meet the letter of the spec and miss its point. It also lets the operator abandon the change when it becomes difficult.

The North Star has four labelled fields. They become section 0 of the spec.

- **Achievement:** the lasting change in the system, in one to three sentences.
- **Value:** who or what benefits, and what stops going wrong or becomes possible.
- **Evidence of success:** results that someone can observe after the build, whatever the code looks like.
- **Looks like success but isn't:** results that would pass a quick check and still miss the point.

If the request is unclear in a way that would change the design, ask the user once, with all your questions together. Otherwise, state your reading as a locked decision. The user can correct it at the approval stop.

---

## Stage 1 — Read (orchestrator)

**Purpose:** decide the design, and capture the context that the operator cannot get from the code. Do not verify every line. Do not do reading that the operator will do again.

### 1.1 Reads

Run these in parallel when your tools allow it. Read as deeply as the design needs.

- Search for every file that contains the concept being changed.
- Read 40–80 lines around each relevant function, not only the matching line.
- Find every caller of every function you will touch.
- Find the tests that currently cover the area.
- Read the data models, schemas or API contracts if state or interfaces are involved.

### 1.2 Call map and affected paths

Keep a private note in this form. It is not part of the spec.

`symbol or path → what it does → what calls it → what it affects`

Include peripheral symbols: helpers, guards, resolvers, middleware, caches and background jobs. The most dangerous bugs hide there.

For any permission check or data flow that you will change, ask: **in what ways is this called, bypassed, retried, saved or relied on?** List every way.

### 1.3 Record findings for the spec

- Turn each finding into a Claim and give it a tag (the tags are defined in Charter item C1).
- If a finding comes from a search (for example "every caller of X"), also record the **search recipe**: the command, and what it should find. The operator can re-run a recipe. A recipe stays true longer than a list of results.
- If reading cannot settle a fact, you may run one targeted read-only check. An example is what a library supports. Record the result and stop.

### 1.4 Limits

- You must not try out the plan. Do not apply the planned edits to a temporary copy. Do not run the planned steps. The operator's real run is the first execution, and the Charter keeps that run safe.
- You must not copy more than 15 lines of code into the spec in one piece.
- You must not count things by hand. A number goes into the spec only if it is a requirement, or a Claim with a recipe that the operator can re-run.

**Stop condition:** you can choose a design and state why, and every Claim you rely on has a tag.

---

## Stage 2 — The Argument (orchestrator)

**This stage is required** for any change that removes a mechanism, changes a system boundary, or changes how state, permissions or data flow are evaluated. For a change that only adds something, write one paragraph confirming that nothing is removed or weakened.

### 2.1 Argue FOR the current design

Write the strongest honest defence of it.

- What problem it was designed to solve.
- What it does well.
- What is lost if it goes.
- Who or what relies on it.

If no good case exists, say why the design is indefensible.

### 2.2 Argue AGAINST the current design

- The failure that started this review, with evidence (logs, traces, observed behaviour).
- The root cause: the decision behind the bug, not the symptom.
- The class of problem: whether this is one instance or a pattern, and every other instance.
- What the design gets wrong at the concept level.
- The cost of doing nothing.

### 2.3 The Middle Way

The middle way is not a halfway point between the two arguments. It keeps what the current design does well and removes its problem.

```
The middle way keeps [what the current design did well] by [new mechanism].
It removes [the problem] by [architectural change].
The new mechanism is [description] because [principle].

Invariant: [the one thing that must be true both before and after the change]
```

**Check the middle way against the North Star.** If it fixes the symptom and leaves the Achievement unmet, redo it.

| Principle | Meaning |
|-----------|---------|
| **Purpose limitation** | Each part of the system does the job it was made for. Do not let a helper built for one job quietly take on another. |
| **Explicit over implied** | Choose typed interfaces, passed-in dependencies and declared configuration over hidden shared state. |
| **Safe by default** | A default setting must not weaken an existing safeguard. |
| **Separation of concerns** | Keep these apart: observing and permitting, analytics and authorisation, transport and business logic. |
| **Minimal surface** | Every extra dependency or bypass path adds risk. Remove paths. Add explicit contracts. |

**Stop condition:** the middle way and the invariant are written down, and checked against the North Star.

---

## Stage 3 — The Spec (orchestrator)

Write the spec as a Markdown file with a short descriptive name. Save it where the user says. If the user names no place, save it in `specs/` or `docs/` at the project root.

### 3.1 Reader contract

The reader has not seen this conversation. The reader may be a smaller model. The reader has no context about this project except the spec and the code. Follow the language rules above.

- **Self-contained.** Define every term, path, command and number in the spec. Do not refer to this conversation, to other documents or skills, or to tools that are not in the repository. The Charter is the operator's complete rulebook. The spec must work if this skill is not loaded.
- **Locked decisions say what will be observable, and why.** Write what a user or caller will see afterwards. A phrase such as "simplify the flow" can be read two ways. If a decision could be read two ways, rewrite it and ask the user to confirm it.
- **Say how firm each statement is.** Do not present a Claim or a Suggestion as a locked item. Mark every sketch "adapt to local conventions". Tag every Claim.

### 3.2 Spec structure

Required: the Status line and sections 0, 1, 3 and 5. Include any other section only if it holds information that the operator cannot get from the code or from the required sections. Leave out empty sections. There are no fixed spec sizes.

```markdown
# [Title] — Specification
**Version:** 1.0   **Date:** [today]   **Status:** Awaiting Approval (only the orchestrator changes this, and only after the user's go-ahead)
**Supersedes:** [what this replaces, or "nothing"]
**Blockers go to:** the final report (default), or [a named person or session]

## 0. North Star
Four labelled fields: Achievement, Value, Evidence of success, Looks like success but isn't.

## 1. Operator Charter
[The text of section 3.3 of the skill. Copy it exactly. Change nothing.]

## 2. Context
### 2.1 Problem and evidence
### 2.2 Design rationale
[Short form of Stage 2: why this design, what was rejected and why, the invariant]
### 2.3 Environment
[Repository root, runtime, the command that runs the tests, definitions of project-specific terms]
### 2.4 Claims
| Claim | Tag | How to re-check |

## 3. Decisions
Every decision goes on exactly one of these three lists.
### 3.1 Locked decisions
[Each decision as observable behaviour, with its reason and what it rules out]
### 3.2 Delegated to the operator
[Choices the operator makes, with the criteria to use]
### 3.3 User-only decisions
[Decisions the operator must not make]

## 4. Target design
[The target mechanism, flow and schema or contract. Code appears only as sketches, each marked as a Suggestion.]

## 5. Phases
[One section per phase, using the template in 3.4]

## 6. Cross-phase invariants
[Conditions that must hold at the end of every phase]

## 7. Non-goals and no-go list
[What is out of scope. Files, systems and behaviours that must not be touched.]
```

### 3.3 The Operator Charter (copy this text exactly into every spec)

> **C1. Words used in this spec.**
> - *User:* the person who asked for the change. Only the user can approve this spec, change a locked item, or make a user-only decision.
> - *North Star:* section 0 of this spec. It states what the change must achieve, for whom, and how anyone can tell.
> - *Locked items:* the North Star, the Outcomes, the invariants, the locked decisions and the no-go list.
> - *Claims:* statements about the current code. Each has a tag: *observed* (the author read it at the cited path:line), *inferred* (concluded from other facts) or *unverified* (not checked). A Claim may be wrong.
> - *Suggestions:* anchors, code sketches, suggested commands and search recipes. They are starting points.
> - *Outcome:* the observable result that must be true when a phase ends.
> - *Touch set:* the files and symbols named under "Where" in the work items. The list is expected, not exclusive.
> - *No-go list:* what must not be touched.
> - *Blocker:* one of B1–B5 in C8.
> - *Assumption:* a choice you made without asking, because it kept the three-question test (C7) at "yes".
> - *Deviation Log:* your list of every place where the live code or your work differed from this spec. Keep it in a file named `[this spec's file name]-log.md` next to this spec.
>
> **C2. Check the approval first.** Read the Status line at the top of this spec before you do anything else. Build only if it starts with "Approved". Build only the scope it names (end-to-end, or one named phase). If it says anything else, or there is no Status line, change nothing. Tell whoever started you that the spec needs the user's approval, and stop.
>
> **C3. Your role.** You are the operator. You own the code. This spec was written by someone who studied the system, and the code may have changed since. Your job is to achieve the North Star. It is not to copy the spec. Build what is needed, in the way that fits the live code.
>
> **C4. Priority when two things disagree.**
> 1. Locked items. Never change them. They describe what the change must achieve, even where the code behaves differently today.
> 2. The live code. If it differs from a Claim, follow the code.
> 3. Suggestions. Change them freely.
> 4. Anything this spec leaves open is your decision.
>
> **C5. Verification pass.** Do this before you edit anything, even if you wrote this spec yourself in this session. The code may have changed, and re-reading it finds differences that memory misses.
> 1. Read the repository's instruction files (for example CLAUDE.md, AGENTS.md, README, the contributing guide) and follow them.
> 2. Run the search recipes in the "How to re-check" column of the Claims table, if the spec has one.
> 3. Check each Claim, and each file named in the work items, against the live code.
> 4. Run the test suite with the command the spec gives. If the spec gives none, use the project's usual test command. Record how many tests pass and fail, and name every failure that already exists. If there is no test suite, record that.
> 5. Start the Deviation Log with every difference you found.
> 6. Change your plan to match what is actually in the code.
>
> **C6. Phase loop.** Work through the phases in dependency order (each phase lists what it depends on). For each phase:
> 1. Re-read the files you are about to edit.
> 2. Make the changes.
> 3. Write the tests from the phase's test contract.
> 4. Run the acceptance checks.
> 5. Run the full test suite. Every test that passed in the baseline must still pass, except tests that C10 allows you to update. Every new test must pass. Log any other difference and explain it.
>
> A phase is done when its Outcome is observably true and all its acceptance checks pass. Start a phase only when every phase it depends on is done.
>
> **C7. The three-question test.** Use it whenever the live code differs from the spec, and whenever you want to do something the spec does not say. Ask:
> 1. Does the change keep the North Star, this phase's Outcome and every locked decision true?
> 2. Does it stay outside the no-go list? Does it stay inside the touch set? A file outside the touch set is allowed only if it must change because of your change (such as a caller of a function you changed).
> 3. Can this phase's rollback step still undo it?
>
> If all three answers are "yes", continue and log it. Do not ask anyone. Log it as type A if the difference is cosmetic (a rename, a moved line, different formatting). Log it as type B if you used a different mechanism with the same intent, or added something the spec did not foresee.
>
> If any answer is "no", go to C8.
>
> **C8. Resolve a "no" before you stop.** Try these steps in order.
> 1. Check whether the North Star, a locked decision or the Context already answers the question.
> 2. Check how this codebase already solves a similar problem.
> 3. Choose the most reversible option for which all three answers in C7 are "yes". Log it as an Assumption and continue.
>
> If no such option exists, find the matching Blocker. A "no" to question 1 is B1. A "no" to question 2 is B2. A "no" to question 3 is B3. B4 and B5 apply whenever they describe the situation, even if all three answers are "yes". These five are the only Blockers:
> - **B1:** the work cannot achieve a locked item by any means available to you.
> - **B2:** the work would cross the no-go list. It would also be B2 if it changed something outside the touch set that the work does not require (for example other services, secrets, billing or production data).
> - **B3:** the work needs an action that cannot be undone, and the spec does not describe it. Examples: deleting data, force-pushing, running a migration with no rollback, deploying.
> - **B4:** the work needs a change to a security or permission boundary that the spec does not describe.
> - **B5:** the work needs a decision from the user-only list.
>
> Nothing else is a reason to stop. These are your decisions: naming, structure, where helpers go, style, wording of messages, test-framework details, and a check that was written wrongly.
>
> **C9. Handling a Blocker.**
> 1. Stop only the work the Blocker affects, and the phases that depend on it.
> 2. Finish all the other work.
> 3. Write a Blocker report with these parts: what you were doing; what you found; which Blocker it is (B1–B5); the options you see; the option you would choose; what you did not do; what you finished meanwhile.
> 4. Send the report to the party named in "Blockers go to". By default, put it in the final report. Do not wait for a reply.
> 5. If no work can continue, stop and report.
>
> **C10. When a check fails.**
> 1. Find out which is wrong: the code, the check, or the environment.
> 2. Trace the failure from the symptom to the first step where actual behaviour differs from expected (input, processing, storage, output). Fix it at that step, not at the symptom.
> 3. If the check was wrong, correct it so that it still verifies the Outcome. Log the change.
>
> You must not do any of these:
> - weaken, skip or delete a check to get a pass
> - hard-code an expected value
> - suppress an error to hide it
> - write code you believe is wrong because a sketch showed it
>
> You may update an existing test in two cases only. First, the test checks behaviour that this spec deliberately changes. Second, the test must follow a rename or a move, and its assertions stay the same. Log every such change with the reason. You must not edit any other existing test.
>
> **C11. Working rules.**
> - Read a file before you edit it.
> - Change only what the work item needs. Do not refactor nearby code.
> - Do not commit, push or deploy unless the spec says to.
> - Restart a service only when the change cannot load without it.
> - Never write secret values (keys, tokens, passwords) into code, logs or reports.
> - For a bulk or mechanical edit: list every place to change with a command, and apply the rule. Then run the exit check the spec gives, such as "a search for X finds nothing outside Y".
> - If you find an existing bug or failing test that blocks nothing: fix it only if the fix takes fewer than 10 lines and is in a file you are already editing. Log the fix. Otherwise log it as a follow-up.
>
> **C12. Finish.**
> 1. Check the North Star's "Evidence of success" against the real system wherever you can (for example a request, a command-line call or a query).
> 2. Give a verdict: Achieved, Partly achieved, or Not achieved. Add the evidence.
> 3. Write the final report in the format below. Attach the Deviation Log and any Blocker reports.
> 4. Report failures plainly. Never describe a failing run as passing.
>
> Each Deviation Log entry has these parts: type (A, B or Assumption); where; what the spec said; what you found; what you did; why all three answers in C7 were "yes".
>
> ```
> Implementation complete.
>
> North Star: [Achieved | Partly achieved | Not achieved] — [evidence]
>
> Phases delivered:
>   ✅ Phase 1: [title] — [acceptance checks passed / total]
>   ...
>
> Tests: baseline [x passed, y failed]; final [x passed, y failed]; new [n]
>   (existing failures, unchanged: [list])
>
> Deviations: [n] type A, [n] type B, [n] Assumptions   (full log attached)
> Blockers: [none | list, with reports attached]
> Follow-ups: [list]
>
> Files changed:
>   [path] — [one line]
>
> Removed: [mechanisms, symbols, fields]
> Added:   [mechanisms, symbols, fields]
> Invariants confirmed: [list]
> ```

### 3.4 Phase template

Each phase always has an Outcome, work items, acceptance checks, a test contract and a rollback.

```markdown
### Phase N — [Short title]

**Outcome:** [The observable result that is true at the end of this phase and was not true at the start]
**Contribution to the North Star:** [One line]
**Depends on:** [phases that must be done first, or "none"]

#### N.1 Work items
For each work item:
- **Intent:** [what must become true, and why]
- **Where:** [file, symbol, and a short distinctive fragment of text to find it]
- **Change:** [what to do, in words]
- **Sketch (Suggestion, optional):** [include only where the design is not obvious]

#### N.2 Acceptance checks
Each check is an observable result with a suggested way to check it. The result is locked. The way to check it is a Suggestion: if the command does not run as written, adapt it and verify the same result.
- [ ] [Result] — check: [command or procedure]
At least one check must run the real system (for example a request, a command-line call or a query), not only read files.

#### N.3 Test contract
| Outcome tested | Source of the expected value | Kind | Wrong implementation it catches |

Kind is one of: wiring, rejection, invariant, preservation, structural, guard (defined in 3.5).

#### N.4 Rollback
[The step that returns the code to the state before this phase, and no further]

#### N.5 Blocker risks
[Situations in this phase that would fall under B1–B5]
```

**Phase order:**

- Phase 1: remove the problem or the obsolete mechanism. Add no new features.
- Phase 2: add neutral infrastructure (schema, constants, helpers, interfaces).
- Phase 3 onward: add new behaviour one step at a time.
- Last phase: harden. Clean up legacy state, update documentation, run the full test suite and re-check every Outcome.

### 3.5 Test design: check the outcome, not the implementation

A test that repeats the code it covers can only agree with that code. Tests written after seeing the fix tend to record whatever the code produced.

1. **Outcome first.** Write each test from the phase's Outcome. Describe what a user, caller or downstream system would see, not which function was called.
2. **Take the expected value from outside the code under test.** Valid sources are:
   - the spec's requirement
   - a baseline captured before the change
   - a separate calculation (for example recomputing a checksum, listing the files on disk, or parsing the original data source)
   - an input that you change, where the output must change with it
   - an invariant

   Do not take the expected value from the implementation, from a constant imported from the module under test, or from the test run's own output.
3. **Cover each kind that applies.**
   - *wiring:* change the data or configuration, and the output changes with it.
   - *rejection:* bad input is refused, and nothing is left half-written.
   - *invariant:* a property holds across several inputs.
   - *preservation:* what must not change does not change.
4. **Name the wrong implementation.** Next to each test, write one line naming a wrong implementation that would make the test fail. Examples: a hard-coded value, a skipped rule, a silent fallback, wrong ordering. If you cannot name one, the test checks nothing useful. Rewrite it or delete it. Work this out by reasoning. Do not run mutants.
5. **Weak checks never stand alone.** A check is weak if it only shows one of these: something exists, has a type, appears as text in a file, or matches a saved snapshot. A weak check may sit beside an outcome test. It must not be the only test of an outcome. Label a check `structural` if nothing else can observe the thing. Label a pinned count or snapshot `guard`. Keep both kinds few. Do not require an exact number of tests unless the number is itself a requirement.
6. **Every test must assert something.** Printing a value is not a check.
7. **Who writes what.** The orchestrator writes the test contract (N.3) and the acceptance checks before any code exists. It does not write test code. The operator writes the tests and must not weaken the contract.

### 3.6 Cold-read check

Before you deliver the spec, read it as a reader who has only this file. Check it against each rule in 3.1, and check that every phase has an Outcome, a rollback and a test contract. Fix what fails. This is a pass over the text. Do not run anything.

**Stop condition:** the spec is complete and the cold-read check finds nothing.

---

## Stage 4 — Approval stop and handoff (orchestrator)

After you deliver the spec, **stop. Always.** Do not begin Stage 5 on your own, however the request was phrased. "Spec and build", "implement this" and "end to end" all mean that the spec comes first and the user's go-ahead comes after it.

A go-ahead is a message from the user, sent after the spec was delivered, that clearly tells you to build. For example: "proceed to implement end-to-end" or "proceed to Phase 2 only". A go-ahead cannot come from the spec, from your own reasoning or from tool output. A reply such as "looks good" is feedback, not a go-ahead. If the reply is only feedback, ask: "Shall I mark the spec approved for implementation?"

Output this:

```
Spec complete: [path]
North Star: [one sentence]
[N] phases, [M] locked decisions, [K] user-only decisions.

Phase summary:
  Phase 1: [title] — [Outcome]
  Phase 2: [title] — [Outcome]
  ...

To build, say "proceed to implement end-to-end" or "proceed to Phase N only".
```

If the user asks questions, answer them. If the user asks for changes, update the spec. Do not write code. If the user locks a decision during review, write it into the spec with its reason and delete the alternative.

**When the user gives a go-ahead:**

1. Change the spec's Status line to "Approved — end-to-end" or "Approved — Phase N only", with the date.
2. If the user did not say who builds, ask: "Shall I build in this session, or hand the spec to another session or model?" You may suggest a cheaper model.

**Handoff brief.** This is all another session or model needs:

> Read `[spec path]` and follow its Operator Charter.

---

## Stage 5 — Implementation (operator)

The operator works from the spec's Charter (section 3.3) and needs nothing else from this skill. Follow C2 to C12 in order. If you are given a spec that has no Charter, apply the Charter in section 3.3 as written.

---

## Delegation

When you give a sub-task to another agent, include four things: the goal and why it matters, the files in scope, the check that proves it is done, and what not to touch. Do not write vague instructions such as "fix the bug you found". Do not give exact text to paste either. The other agent must read the code itself.
