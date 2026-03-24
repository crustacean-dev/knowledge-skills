---
name: tasks
description: Break a feature into ordered implementation tasks. Use this skill whenever the user says "tasks", "tasks <feature>", "break this down", "what needs to be built", "implementation order", "task list", "what's next", "story breakdown", or wants to know what to implement and in what order. Also use when the user wants to update task status ("mark TASK-3 done"), reorder tasks, add tasks, split tasks, or check progress on existing tasks. Trigger even for casual phrasing like "what's left to build", "figure out the implementation steps", "create stories for this feature", or "what do I need to build first". Produces .knowledge/features/<feature>/tasks.md from spec.md + plan.md. Does NOT produce specs (/spec), plans (/plan), code (/implement), or review specs (/clarify).
---

# Tasks

You produce `.knowledge/features/<feature>/tasks.md` — an ordered list of implementation tasks that `/implement` will execute one by one. Each task is a self-contained unit of work with no ambiguity about what to build.

## Prerequisites

Both must exist:

- `.knowledge/features/<feature>/spec.md` — what to build
- `.knowledge/features/<feature>/plan.md` — how to build it

If either is missing, STOP. Tell the user which one to create first (`/spec <feature>` or `/plan <feature>`).

Also read `.knowledge/constitution.md` and `.knowledge/architecture.md` for project-level context.

## Phase 0: Detection

Check if `.knowledge/features/<feature>/tasks.md` already exists.

- **If it exists:** Read it in full. You are in **amendment mode**. Ask the user what they want to change — add tasks, reorder, update status, split a task, remove a task. Preserve completed tasks and their status. Do NOT regenerate the full list.
- **If it does not exist:** Proceed to Phase 1.

## Phase 1: Analyze

Read `spec.md` and `plan.md` in full. Identify:

1. **Implementation units** — What are the distinct pieces of work? Each user story may map to one or more tasks. Each architectural component from the plan may need its own task.
2. **Dependencies** — What must be built before what? A service that depends on a data model means the data model task comes first.
3. **Foundational tasks** — Is there setup work (scaffolding, config, types) that other tasks depend on? These come first.
4. **Test tasks** — Are tests a separate task or part of each task? Default: tests are part of each task (each task includes writing tests for what it builds). Only split tests into a separate task if the spec has a complex test setup (fixtures, mocks, test utilities) that other tasks depend on.

## Phase 2: Draft

Generate the task list using the template below. Present it to the user for review.

**Sizing rules:**

- A task should be completable in a single `/implement` session — typically 1-3 files created or modified.
- If a task touches more than 5 files, it's too big. Split it.
- **Maximum 3 acceptance criteria per task.** Each AC becomes a test assertion that `/implement` must satisfy — more than 3 means the task is doing too much and should be split. This is a hard limit, not a suggestion.
- If describing the task takes more than 10 lines, it's too big.

**Right-sized AC example:**

```
**Acceptance criteria:**
- Returns `TokenPayload` for a valid, non-expired token
- Returns `null` for expired, tampered, or revoked tokens — does not throw
- Tests pass for valid, expired, tampered, and revoked token scenarios
```

Three criteria: the happy path, the failure path, and tests. That's the right shape. If you find yourself writing a 4th criterion, it means the task covers too many concerns — split it. For example, "validates token" and "checks revocation list" are two tasks if they each need their own AC.

**Ordering rules:**

- Dependencies first. If task B imports from task A's output, A comes first.
- Foundational tasks (types, interfaces, config) before logic tasks.
- Core logic before edge cases and error handling.
- Happy path before error paths.

## Phase 3: Validate & Write

Iterate until the user confirms, then write to `.knowledge/features/<feature>/tasks.md`. Create the directory if needed.

## Template

```markdown
# Tasks — <Feature Name>

> Feature: `.knowledge/features/<feature>/spec.md`
> Plan: `.knowledge/features/<feature>/plan.md`
> Total: <N> tasks

---

## Progress

| Status      | Count |
| ----------- | ----- |
| TODO        | <n>   |
| IN_PROGRESS | <n>   |
| DONE        | <n>   |

---

## Task list

### TASK-1: <Title>

**Status:** TODO
**Depends on:** — (none, or TASK-N)
**Implements:** US-1 (partial), US-2
**Files:**

- Create `<path/to/file>` — <what this file does>
- Modify `<path/to/file>` — <what changes>

**Description:**
<2-5 sentences: what to build, what decisions are already made in plan.md, what to watch out for.>

**Acceptance criteria:**

- <Criterion — verifiable, from spec.md or derived from plan.md>
- <Criterion>
- Tests pass for <what's being tested>

---

### TASK-2: <Title>

**Status:** TODO
**Depends on:** TASK-1
**Implements:** US-3
**Files:**

- Create `<path/to/file>`
- Modify `<path/to/file>`

**Description:**
<...>

**Acceptance criteria:**

- <...>

---

### TASK-N: <Title>

...
```

## Status lifecycle

```
TODO → IN_PROGRESS → DONE
```

- `/implement` sets a task to IN_PROGRESS when starting it, DONE when finished.
- If a task is blocked, add a `**Blocked by:** <reason>` field.
- Tasks can go back to TODO if implementation reveals the task was wrongly scoped.

## Rules

- This skill produces ONLY `tasks.md` for a feature. Never specs, plans, or code.
- Every task must reference which US it implements via `**Implements:** US-N`. Orphan tasks (not tied to any US) are not allowed — if a task is needed but no US covers it, flag it and ask the user if a US is missing from the spec.
- Every task must list the **exact files** to create or modify. "/implement should figure out where to put it" is not acceptable.
- Tasks must be **ordered** — the list is a sequence, not a bag. Dependencies are explicit.
- No task should depend on more than 2 other tasks. If it does, there's a missing intermediate task.
- Each task includes writing tests for what it builds. Tests are not a separate task unless the spec requires shared test infrastructure.
- Do NOT duplicate spec content in task descriptions. Reference the US ("Implements US-3") and add only implementation-specific guidance from plan.md.
- If the plan or spec has unresolved questions ouvertes, flag them as blockers on the affected tasks.
