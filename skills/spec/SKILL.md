---
name: spec
description: Create or amend feature specifications with typed code examples, verifiable acceptance criteria, and explicit edge cases. Produces .knowledge/features/<feature>/spec.md — the definitive WHAT document for a feature. ALWAYS use this skill when the user wants to define what a feature does before building it, whether they say "spec", "specify", "requirements", "user stories", "acceptance criteria", "what should this do", "define the feature", "expected behavior", "edge cases for", or any variant asking about feature behavior, inputs/outputs, or success criteria. Also trigger when the user wants to amend an existing spec, add user stories, refine acceptance criteria, or define what's in/out of scope for a feature. This skill describes WHAT to build (behavior, contracts, verification) — never HOW (architecture, algorithms, data structures → use /plan), never task breakdowns (→ /tasks), never code (→ /implement), never project principles (→ /constitution).
---

# Spec

You produce `.knowledge/features/<feature>/spec.md` — the definitive document for **what** a feature does, **who** it's for, and **how to verify** it works. The spec describes behavior, not implementation. `/plan` handles the how.

## Prerequisite

If `.knowledge/constitution.md` does not exist, STOP. Tell the user to run `/constitution` first.

## What vs. How

This boundary is the spec's most important discipline. If the user provides implementation details, acknowledge them but redirect — they belong in `/plan`.

**What (belongs here):**
- "The router matches HTTP requests to handlers based on URL patterns" — observable behavior
- "Returns `{ valid: false, error: 'EXPIRED' }` when the token is past expiry" — input/output contract
- "Compile-time error if handler accesses a parameter not in the pattern" — verifiable criterion

**How (redirect to /plan):**
- "Use a trie data structure for matching" — internal data structure
- "Implement Aho-Corasick for multi-pattern matching" — algorithm choice
- "Store routes in a radix tree with compressed nodes" — implementation architecture
- "O(n) complexity where n is path length" — performance of an implementation

When the user mixes what and how, say so directly: *"The trie, radix tree, and algorithm choices are /plan concerns — they describe how to build the router, not what it does. What does the router do from the developer's perspective?"*

## Phase 0: Detection

Check if `.knowledge/features/<feature>/spec.md` already exists.

- **If it exists:** Read it in full. You are in **amendment mode**. Also read `.knowledge/constitution.md` to check compatibility. Ask the user what they want to change. Propose surgical amendments — section by section, showing current vs. proposed. Preserve unchanged sections verbatim. Do NOT restart the interview. Add new content (user stories, edge cases, hors scope items) alongside existing content. If the amendment contradicts the constitution, flag it before proceeding.
- **If it does not exist:** Proceed to Phase 1.

## Phase 1: Extract & Discover

Read `.knowledge/constitution.md` and `.knowledge/architecture.md` (if it exists) for context. Extract constraints and principles that apply to this feature.

Parse the user's initial message for information already provided. Map it against the categories below — acknowledge what you understood, ask follow-ups for gaps, skip what's answered.

Then interview the user. ONE question at a time.

**Questions (ask only what's missing):**

1. **Purpose** — "What does this feature do? What problem does it solve for the user?"
2. **Users** — "Who uses this feature? What's their context? (developer using an API, end user in a browser, CI pipeline, etc.)"
3. **Behavior** — "Walk me through the main scenario. What does the user do, and what happens?"
4. **Inputs/Outputs** — "What goes in? What comes out? Be specific — types, formats, examples."
5. **Variations** — "What are the alternative paths? What if the input is invalid? What if a dependency fails?"
6. **Boundaries** — "What does this feature explicitly NOT do?"

**Pushback is mandatory.** The spec must contain typed code examples with concrete inputs and outputs. Vague answers cannot become verifiable acceptance criteria.

- "You said 'it handles errors' — which errors? What does the user see for each one? What's the error format?"
- "'It processes the data' — what data? What type? What's an example input and the exact expected output?"
- "'The usual CRUD operations' — list them. For each one: what's the input, what's the output, what can go wrong?"
- "'It should be fast' — that's a principle, not a behavior. What's the observable behavior? Does the user see a loading state? A progress indicator? Nothing?"
- "'It works like X' — show me. Write the code the user would write to use this feature."

### Proactive proposals

Sometimes the user doesn't have an answer — they say "you decide", "I'm not sure", "what do you think?", "fill it in", or simply don't know. Don't stall the interview. Instead, propose an answer grounded in existing `.knowledge/` context.

Before proposing, read what's available: `constitution.md`, `architecture.md`, and any existing specs in `.knowledge/features/`. Base your proposal on constraints, principles, patterns, or precedents you find there. If nothing in `.knowledge/` supports a reasonable proposal, say so — do not guess.

**Format every proposal like this:**

> **Proposal:** <your suggested answer>
> **Reasoning:** <which `.knowledge/` source supports this and why — cite the file and the specific principle, constraint, or pattern>
> **Confirm, adjust, or reject?**

The user's decision is final. If they confirm, treat it as their answer and move on. If they adjust, use their version. If they reject, ask the question again or try a different angle.

**Examples:**

- User says "I don't know what errors to handle — you decide."
  → Read `constitution.md` for error-handling principles. If it says "all public APIs return typed Result objects", propose specific error variants based on that pattern, citing the principle.

- User says "what do you think the output format should be?"
  → Check existing specs in `.knowledge/features/` for output format precedents. If `router/spec.md` returns `{ success: true, data: T }`, propose the same shape for consistency, citing that spec.

- User says "fill in the edge cases."
  → Derive edge cases from the input types and constraints already discussed, plus any relevant interdits from the constitution. Cite each source.

Do NOT proceed to Phase 2 with vague answers.

## Phase 2: Draft

Generate the spec using the template below. Present it to the user for review. Do NOT write to disk yet.

**Critical rule:** Every User Story must include a **typed code example** showing the feature being used. Not pseudo-code — real TypeScript (or the project's language) that a developer could read and understand the expected behavior from. If the feature is not code-facing (UI, CLI output), show the exact input/output exchange instead.

## Phase 3: Validate & Write

Iterate until the user confirms, then write to `.knowledge/features/<feature>/spec.md`. Create the directory if needed.

## Template

````markdown
# Spec — <Feature Name>

> Package: `<package name if applicable>`
> Feature: <short identifier>

---

## Objective

<1-2 sentences: what this feature does and why it matters. Tied to a user need, not an implementation detail.>

---

## Constraints

> From constitution.md — which principles and interdits apply to this feature.

- <Constraint 1 — e.g. "Zero runtime overhead (Principle #2)">
- <Constraint 2 — e.g. "No reflect-metadata (Interdit)">

---

## User stories

### US-1: <Title>

> As a <role>, I want <action> so that <benefit>.

**Input:**

```typescript
// Exact input — types, structure, example values
<code>
```

**Output:**

```typescript
// Exact output — what the user gets back
<code>
```

**Acceptance criteria:**

- <Criterion — verifiable, not subjective>
- <Criterion>

### US-2: <Title>

> As a <role>, I want <action> so that <benefit>.

**Input:**

```typescript
<code>
```

**Output:**

```typescript
<code>
```

**Acceptance criteria:**

- <Criterion>

---

## Edge cases

> What happens when things go wrong or inputs are unexpected. Each edge case must specify the input, the expected behavior, and the error (if any).

### EC-1: <Description>

**Input:** <what triggers this edge case>
**Expected behavior:** <what should happen — error message, fallback, rejection>

### EC-2: <Description>

**Input:** <trigger>
**Expected behavior:** <outcome>

---

## Hors scope

> What this feature explicitly does NOT cover. Prevents scope creep during implementation.

- <Thing this feature won't do — and why if not obvious>
- <Thing>

---

## Questions ouvertes

> Unresolved points that need a decision before or during implementation.

- <Question — with context on why it matters>
````

## Rules

- This skill produces ONLY `spec.md` for a feature. Never architecture, plans, tasks, or code.
- Every user story MUST have typed Input/Output code examples. "The function returns the result" is not an output. `{ success: true, data: User[] }` is.
- Acceptance criteria must be **verifiable** — a test could be written from them. "It works correctly" is not a criterion. "Returns a 404 with `{ error: 'NOT_FOUND', message: 'User {id} does not exist' }` when the user ID doesn't match any record" is.
- Edge cases must specify **exact input and exact expected behavior**. "Handles bad input gracefully" is not an edge case.
- Constraints must reference specific principles or interdits from the constitution by name.
- Hors scope must be **explicit** — if a related behavior is not covered, say so. This prevents `/implement` from guessing.
- If the feature contradicts the constitution, flag it and ask the user to resolve the conflict before proceeding.
- The spec describes **what**, not **how**. If you catch yourself writing implementation details (algorithms, data structures, internal architecture), stop — that belongs in `/plan`.
