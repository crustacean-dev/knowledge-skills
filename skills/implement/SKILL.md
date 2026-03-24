---
name: implement
description: Write code and tests for a single task from .knowledge/features/<feature>/tasks.md. Picks the next TODO task (or a specific TASK-N), reads its acceptance criteria, referenced user stories, plan, architecture, and conventions, then writes exactly the code and tests needed to satisfy the ACs. Updates task status to DONE when complete. Use this skill whenever the user wants to write code for a planned task — trigger phrases include "implement", "implement auth", "implement TASK-3", "build this", "code this task", "start building", "next task", "what's next to implement", "work on TASK-5", "implement the next task", or any variant asking to write code against a task breakdown. Also trigger for casual phrasing like "ok build it", "let's code the first task", "I finished planning, now write the code", or "pick up where we left off". This skill handles ONE task per invocation — it does NOT write specs (/spec), create plans (/plan), break down tasks (/tasks), review code (/review), challenge implementations (/challenge), do general refactoring, or add tests to existing code outside a task context. If the user asks to "add tests" or "refactor" without referencing a task from tasks.md, this skill should NOT trigger.
---

# Implement

You execute a single task from `.knowledge/features/<feature>/tasks.md`. Your scope is the task's acceptance criteria — nothing more, nothing less. You are a disciplined implementer, not a creative architect.

## Prerequisites

All must exist:

- `.knowledge/constitution.md` — principles and interdits
- `.knowledge/architecture.md` — project structure and conventions
- `.knowledge/conventions.md` — code style and testing rules
- `.knowledge/features/<feature>/spec.md` — what the feature does
- `.knowledge/features/<feature>/plan.md` — how to build it
- `.knowledge/features/<feature>/tasks.md` — task breakdown

If any are missing, STOP. Tell the user which to create first. Be specific — if tasks.md is missing, suggest `/tasks <feature>`. If spec.md is missing, suggest `/spec <feature>`.

## Phase 0: Resolve feature and select task

### Step 1: Resolve the feature

The user's input may refer to a feature in various ways. Resolve it to a `.knowledge/features/<feature>/` directory:

- **Explicit feature name** ("implement auth", "implement the authentication feature") → Map to `.knowledge/features/auth/`.
- **Explicit task ID** ("implement TASK-3", "work on TASK-5") → The feature must be inferred. If only one feature directory has a `tasks.md`, use it. If multiple exist, ask: "Which feature is TASK-3 from? I see tasks for: auth, compiler, router."
- **No feature specified** ("next task", "what's next to implement") → Scan `.knowledge/features/*/tasks.md` for features with TODO tasks. If exactly one feature has TODO tasks, use it. If multiple features have TODO tasks, ask: "Multiple features have pending tasks. Which one? [list them with TODO counts]." If no features have TODO tasks, tell the user everything is complete.

### Step 2: Select the task

Once the feature is resolved:

- **User specifies a task** ("implement TASK-3", "work on TASK-3") → Use that task.
- **User specifies only a feature** ("implement auth") or says "next task" → Read `tasks.md`, find the first TODO task whose dependencies are all DONE. If no TODO task has satisfied dependencies, tell the user which tasks are blocked and why.
- **All tasks are DONE** → Tell the user: "All N tasks for <feature> are complete. Nothing to implement."

### Step 3: Validate the selected task

Read the selected task. Verify:

1. **Status is TODO.** If DONE, ask "TASK-N is already done. Do you want to redo it?" If IN_PROGRESS, ask "TASK-N is marked as in progress. Do you want to continue or restart it?"
2. **All dependencies are DONE.** If not, tell the user exactly which tasks must be completed first: "Cannot implement TASK-5 — it depends on TASK-4, which is still TODO."

## Phase 1: Context loading

Load ONLY what this task needs. Do not read the entire codebase.

1. **Task definition** — the full TASK-N block from tasks.md
2. **Referenced user stories** — read only the US-N sections listed in `**Implements:**`
3. **Plan.md** — read the sections relevant to this task (approach, data model, integration, API surface, decisions)
4. **API contracts** — read `api/*.d.ts` files if the task involves a public API
5. **Conventions** — read conventions.md for code style, testing, naming
6. **Architecture** — read architecture.md for directory structure and module boundaries
7. **Existing code** — read only the files listed in the task's `**Files:**` section that already exist (to modify)

## Phase 2: Implement

First, update `tasks.md` — set the task's status from TODO to IN_PROGRESS and update the Progress table counts. This signals that work has started.

Then write the code. Follow these rules strictly:

### Scope discipline

- Implement ONLY what the acceptance criteria require. Do not add features, optimizations, or "nice to have" code.
- If the task says "Create `src/auth/token.ts`", create that exact file in that exact path. Do not rename, relocate, or reorganize.

### Gaps in spec or plan

If you discover something missing from the spec or plan during implementation:

- **Minor gap** (e.g., a missing error message string, an unspecified default value) — make a reasonable choice, document it in the summary Notes section, and continue. Do not block the task for trivia.
- **Major gap** (e.g., a missing data model field that changes the API contract, an unresolved question ouverte that affects behavior) — flag it as blocked. Do NOT invent behavior. Add a note explaining what's missing and what decision is needed.

### Code rules

- Follow `conventions.md` for all code style decisions (naming, imports, error handling, file structure).
- Follow `architecture.md` for directory structure and module boundaries.
- Follow `constitution.md` for principles and interdits — if the implementation would violate an interdit, STOP and flag it.
- Match the types in `api/*.d.ts` contracts exactly. If the contract says `string | null`, do not return `string | undefined`.

### Tests

- Write tests as part of the task (unless tasks.md explicitly says otherwise).
- Each acceptance criterion maps to at least one test assertion. If a task has more than 3 AC (which can happen for tasks created before the max-3 rule), cover them all — don't skip ACs just because there are many.
- Follow the testing conventions in `conventions.md` (framework, file naming, mocking approach).
- Tests must pass. If a test fails, fix the implementation — not the test (unless the test is wrong, in which case flag it).

### What NOT to do

- Do NOT refactor existing code that isn't in the task's `**Files:**` list. Exception: updating the package's `index.ts` barrel export to re-export newly created modules is allowed — it's part of making the new code usable.
- Do NOT update dependencies or install packages unless the task explicitly requires it.
- Do NOT modify specs, plans, or other tasks. If something needs to change, tell the user.
- Do NOT implement the next task. One task per `/implement` invocation.

## Phase 3: Verify

After writing the code:

1. **Run tests** if possible (check conventions.md for the test command).
2. **Run linting** if possible (check conventions.md for the lint command).
3. **Check each acceptance criterion** — for each one, confirm it's satisfied and point to the code/test that satisfies it.

Present a summary using this exact format — the ✅ checkmarks and `file:line` references matter because downstream skills parse this output:

```
## TASK-N: <Title> — Implementation complete

### Acceptance criteria
- ✅ <Criterion 1> — `src/auth/token.ts:42`, test: `token.spec.ts:15`
- ✅ <Criterion 2> — `src/auth/token.ts:67`, test: `token.spec.ts:38`
- ✅ <Criterion 3> — tests pass (3/3)

### Files created/modified
- Created `src/auth/token.ts`
- Created `src/auth/__tests__/token.spec.ts`
- Modified `src/auth/index.ts` — added export

### Notes
- <Any observations, discovered issues, or suggestions for future tasks>
```

Use ✅ (the checkmark emoji), not dashes or other markers. Each criterion must reference the source file and line number where it's satisfied, and the test file and line where it's tested.

## Phase 4: Update status

After the user confirms the implementation is acceptable:

1. Update `tasks.md` — set the task's status from IN_PROGRESS to DONE.
2. Update the Progress table counts.
3. If this was the last task, note that all tasks for this feature are complete.

If the user wants changes, make them and re-verify before updating status.

## Rules

- ONE task per invocation. Never implement multiple tasks in a single session.
- Scope is the acceptance criteria. Not more, not less. If an AC says "returns null for expired tokens", implement exactly that — don't add logging, metrics, or retry logic.
- Files listed in the task are the ONLY files you may create or modify (plus their test files). Touching other files is a scope violation.
- If a constitution interdit would be violated, STOP. Do not implement a workaround — flag it.
- If the plan's data model doesn't match the spec's expected output, flag the inconsistency. Do not guess which one is right.
- Tests are mandatory. A task is not DONE without passing tests.
- If you can't satisfy an AC because of a major gap in the spec or plan, flag it as blocked and explain what's missing. Do not invent behavior.
