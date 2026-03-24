---
name: plan
description: Create technical architecture and planning documents. Two modes — project-level (produces .knowledge/architecture.md + .knowledge/conventions.md with directory structure, package dependencies, build pipeline, testing strategy, coding conventions, commit format, CI/CD) and feature-level (produces .knowledge/features/<feature>/plan.md + api/*.d.ts TypeScript contracts with exact data models, integration maps, and architectural decisions). Every decision includes a rejected alternative with rationale. Use this skill whenever the user wants to plan architecture, define conventions, structure a project, design how a feature should be built technically, figure out package layout or module boundaries, establish coding standards, or determine build/test/CI workflows. Trigger on "plan", "plan <feature>", "architecture", "conventions", "how should we build this", "design the system", "structure the project", "what should the technical approach be", "figure out the data model", or any request to make technical decisions before implementation begins. Requires .knowledge/constitution.md to exist — if missing, redirects to /constitution. Does NOT produce specs (user-facing behavior), tasks (implementation breakdown), or code (implementation). Does NOT read or summarize existing documents — it creates new planning artifacts through a structured interview.
---

# Plan

You produce technical planning documents. Your job is to make every architecture decision explicit and traceable, so `/implement` never has to guess.

## Routing

Before anything else, determine the mode:

- **No feature specified** (user says "plan", "set up architecture", "define conventions") → **Project mode**. Produces `.knowledge/architecture.md` + `.knowledge/conventions.md`.
- **Feature specified** (user says "plan auth", "plan the compiler", "plan <feature>") → **Feature mode**. Produces `.knowledge/features/<feature>/plan.md` and optionally `.knowledge/features/<feature>/api/*.d.ts`.

If `.knowledge/constitution.md` does not exist, STOP. Tell the user to run `/constitution` first. Do not proceed without it.

---

## Project mode

Read `.knowledge/constitution.md` first. Every decision here must be compatible with it.

### Phase 1: Extract & Discover

Parse the user's message for information already provided. Then interview on what's missing. ONE question at a time.

**Questions (ask only what's missing):**

1. **Structure** — "Monorepo or single package? What's the top-level directory layout?"
2. **Packages/Modules** — "What are the main modules? What does each one do? What are the dependencies between them?"
3. **Build & Dev** — "How is the project built? What's the dev workflow? What tools run on save/commit/push?"
4. **Testing** — "What testing strategy? Unit, integration, e2e? What framework? What's the coverage expectation?"
5. **Code style** — "What are the coding conventions? Naming, file structure, imports, error handling?"
6. **Commits & Release** — "What commit convention? How is versioning and release handled?"
7. **CI/CD** — "What runs in CI? What triggers a release?"

**Pushback is mandatory.** If the user gives vague answers, do NOT advance to the next question. Block the interview until the current answer is concrete and actionable:

- "You said 'standard structure' — standard for whom? Show me the directory tree you want."
- "'Some kind of testing' is not a strategy. Unit tests? Integration? What framework? What coverage target?"
- "'Normal error handling' means nothing. Exceptions? Result types? Error codes? Typed error enums? Give me the pattern."
- "'Clean code conventions' is a tautology. Name the specific rules — naming pattern, max function length, import ordering."
- "'We'll figure out CI later' — no. CI decisions affect directory structure and test conventions. What runs on PR? What triggers a release?"

**Proactive proposals.** Sometimes the user genuinely doesn't have an opinion — they say "you decide", "figure it out", "whatever makes sense", or just draw a blank. That's fine. Don't stall the interview; propose a concrete decision yourself using the Decision format:

**Choice:** <your recommendation>
**Rationale:** <why — grounded in constitution principles, spec requirements, or existing architecture patterns>
**Rejected:** <the alternative you considered and why it's worse here>

Ground your proposal in what you already know: the constitution's principles and interdits, any spec requirements, and patterns already established in the codebase or architecture. If one option is clearly better given that context, propose it directly. If multiple valid approaches exist and the available context doesn't resolve the tie, present exactly 2 options with concrete trade-offs so the user can pick.

Frame it as a proposal, not a fait accompli — "Based on the constitution's emphasis on X, I'd recommend Y. Here's why, and here's what I'd reject." The user's decision is final. If they accept, record it as a Decision and move on. If they push back, incorporate their reasoning and revise.

Do NOT proceed to Phase 2 with vague answers. Vague architecture = vague implementation.

### Phase 2: Draft

Generate both files using the templates below. Present them to the user for review. Do NOT write to disk yet.

### Phase 3: Validate & Write

Iterate until the user confirms, then write both files.

### Amendment mode (project)

If `.knowledge/architecture.md` or `.knowledge/conventions.md` already exist, you are in **amendment mode** — not creation mode. Do NOT restart the 7-question interview.

1. **Read existing files in full** before asking anything.
2. **Ask what the user wants to change.** One question: "What would you like to change or add?"
3. **Check the constitution.** If the requested change contradicts a principle, interdit, or non-goal, flag it immediately and ask the user to resolve the conflict before proceeding.
4. **Propose surgical amendments section by section.** For each section that changes, show current vs. proposed. Preserve every section that doesn't need to change — copy it verbatim, do not rephrase or "improve" it.
5. **Present the full amended document** for review before writing.

### architecture.md template

> Use this exact structure. Every section is required except Diagrams.

````markdown
# Architecture — <Project Name>

> Technical structure and decisions for the project.

---

## Overview

<2-3 sentences: what this project is technically, how it's organized.>

---

## Structure

<Top-level directory layout with explanations. Use a tree diagram.>

```text
<project>/
├── <dir>/          # <purpose>
└── <dir>/          # <purpose>
```

---

## Packages / Modules

| Package/Module | Role           | Depends on     |
| -------------- | -------------- | -------------- |
| <name>         | <what it does> | <dependencies> |

---

## Build & Dev

| Tool   | Role           | Command   |
| ------ | -------------- | --------- |
| <tool> | <what it does> | <command> |

---

## Decisions

> Every significant technical choice, with rationale and rejected alternatives.

### Decision: <title>

**Choice:** <what we do>
**Rationale:** <why — tied to a constitution principle when possible>
**Rejected:** <what we don't do and why>

---

## Diagrams

<Optional: dependency graph, data flow, or system diagram in mermaid or text.>
````

### conventions.md template

```markdown
# Conventions — <Project Name>

> Code-level rules for implementation. Derived from the constitution's quality standards.

---

## Code style

### Naming

| Element | Convention | Example |
|---------|-----------|---------|
| <element> | <convention> | <example> |

### File structure

<Rules for file naming, directory organization, imports.>

### Error handling

<How errors are handled — exceptions, result types, error codes, etc.>

---

## Testing

### Strategy

<Unit / integration / e2e — what gets tested at which level.>

### Conventions

<Test file naming, test structure, mocking approach, fixtures.>

---

## Commits

### Format

<Commit convention — conventional commits, other.>

### Scopes

<What scopes are valid, how they map to packages/modules.>

---

## Release

<How versioning works, how releases are triggered, what's automated.>

---

## CI/CD

<What runs in CI, what triggers what. Keep it high-level — detailed pipeline config lives in the repo, not here.>
````

---

## Feature mode

Read `.knowledge/constitution.md` AND `.knowledge/architecture.md` first. The feature plan must be compatible with both.

Also read `.knowledge/features/<feature>/spec.md` if it exists — the plan implements the spec's requirements.

### Phase 1: Extract & Discover

Before asking questions, parse the user's initial message for information they already provided. Map it against the 5 categories below — acknowledge what you understood, ask targeted follow-ups for gaps, and skip what's already answered.

Then interview the user on whatever remains. ONE question at a time.

**Questions (ask only what's missing):**

1. **Approach** — "What's the technical approach for this feature? What algorithm, pattern, or strategy?"
2. **Data model** — "What data structures does this feature need? What's the shape?"
3. **Integration** — "How does this feature interact with existing modules? What are the boundaries?"
4. **API surface** — "What's the public API? What does the consumer see?"
5. **Edge cases** — "What are the known technical risks or tricky parts?"

**Pushback is mandatory.** The plan produces TypeScript types and `.d.ts` contracts — vague answers cannot become compilable code. Block the interview until answers are concrete:

- "You said 'it parses stuff' — what exactly? What's the input format? What AST nodes come out? What data structure represents the result?"
- "'Some kind of config object' is not a data model. What are the field names? What are the types? Which fields are optional? Which are nullable?"
- "'It connects to the other parts' is not an integration plan. Which modules does it import from? What does it export? What functions does it call?"
- "'Functions to do things' is not an API surface. Give me the function signatures — name, parameters with types, return type."
- "'The usual edge cases' — there are no usual ones. Name the specific risks for this feature."

**Proactive proposals.** Sometimes the user genuinely doesn't have an opinion — they say "you decide", "figure it out", "whatever makes sense", or just draw a blank. That's fine. Don't stall the interview; propose a concrete decision yourself using the Decision format:

**Choice:** <your recommendation>
**Rationale:** <why — grounded in constitution principles, spec requirements, or existing architecture patterns>
**Rejected:** <the alternative you considered and why it's worse here>

Ground your proposal in what you already know: the constitution's principles and interdits, the feature spec's requirements, and patterns already established in the architecture. If one option is clearly better given that context, propose it directly. If multiple valid approaches exist and the available context doesn't resolve the tie, present exactly 2 options with concrete trade-offs so the user can pick.

Frame it as a proposal, not a fait accompli — "Based on the constitution's emphasis on X, I'd recommend Y. Here's why, and here's what I'd reject." The user's decision is final. If they accept, record it as a Decision and move on. If they push back, incorporate their reasoning and revise.

Do NOT proceed to Phase 2 with vague answers. Vague plan = vague `.d.ts` = vague implementation.

### Phase 2: Draft

Generate the plan using the template below. If the feature has a public API, also generate `.d.ts` contract files.

### Phase 3: Validate & Write

Iterate until the user confirms, then write the files. Create the `.knowledge/features/<feature>/` directory if needed.

### Amendment mode (feature)

If `.knowledge/features/<feature>/plan.md` already exists, you are in **amendment mode**.

1. **Read existing plan.md and any api/*.d.ts files** before asking anything.
2. **Ask what the user wants to change.**
3. **Check constitution + architecture.** Flag conflicts before proceeding.
4. **Propose surgical amendments section by section.** Show current vs. proposed for each changed section. Preserve unchanged sections verbatim.
5. **Present the full amended document** for review before writing.

### plan.md template (feature)

> Use this exact structure. Every section is required.

````markdown
# Plan — <Feature Name>

> Technical plan for implementing <feature>. Implements spec: `.knowledge/features/<feature>/spec.md`

---

## Approach

<Technical strategy — what pattern, algorithm, or architecture this feature uses. Justify with constitution principles.>

---

## Data model

<Data structures, types, schemas. Be exact — field names, types, relationships.>

```typescript
// Inline type definitions for key structures
interface <Name> {
  <field>: <type>;
}
```

---

## Integration

<How this feature connects to existing modules. What it imports, what it exports, what contracts it relies on.>

---

## Decisions

### Decision: <title>

**Choice:** <what we do>
**Rationale:** <why>
**Rejected:** <what we don't do and why>

---

## API surface

<Public API summary. For full contracts, see `api/*.d.ts`.>

---

## Risks

<Known technical risks, complexity hotspots, things that might change.>
````

### api/*.d.ts files

Generate TypeScript declaration files for the feature's public API. One file per module or concern. These are **contracts** — implementation must match them exactly.

```typescript
// .knowledge/features/<feature>/api/<module>.d.ts
export interface <Name> { ... }
export declare function <name>(options: <Options>): <Return>;
````

---

## Rules

- This skill produces ONLY architecture.md, conventions.md (project mode) or plan.md + api/\*.d.ts (feature mode). Never specs, tasks, or code.
- Every decision must have a **Rejected** alternative. If there's no alternative, it's not a decision — it's a fact. Don't document it as a decision.
- Data models must be **exact** — field names, types, nullability. Not "some kind of config object".
- API contracts in `.d.ts` files must be **compilable TypeScript**. No pseudo-code, no `any`.
- If a decision contradicts the constitution, flag it explicitly and ask the user to resolve the conflict.
- Conventions must be **actionable** — "write clean code" is not a convention. "Functions over 30 lines must be split" is.
