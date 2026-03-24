---
name: constitution
description: Create or amend .knowledge/constitution.md — the governing document that defines a project's non-negotiable principles, tech stack (with versions), interdits (what's explicitly forbidden), and quality standards. Use this skill whenever the user wants to start a new project, define or revisit project principles, establish ground rules before implementation, settle recurring technical debates ("should we use an ORM?"), amend an existing constitution, add or remove interdits, or check what the current constitution says. Trigger phrases include "constitution", "new project", "let's start", "define principles", "project setup", "what's forbidden", "ground rules", "project charter", "interdits". Also trigger when no .knowledge/constitution.md exists yet and the user is about to begin spec or implementation work. This skill ONLY produces constitution.md — it does not produce architecture.md, specs, API designs, README files, or implementation code.
---

# Constitution

You produce `.knowledge/constitution.md` — the governing document of the project. Every other skill reads it. Every decision must be compatible with it. Get it right.

## Process

### Phase 0: Detection

Before anything else, check if `.knowledge/constitution.md` already exists.

- **If it exists:** Read it in full. You are now in **amendment mode**, not creation mode. Do NOT restart the 7-question interview. Instead, ask the user what they want to change, flag any conflicts between their request and existing interdits or principles, and propose surgical amendments — section by section, showing current vs. proposed. Only touch sections that need to change. Preserve everything else verbatim.
- **If it does not exist:** Proceed to Phase 1.

### Phase 1: Extract & Discover

Before asking questions, parse the user's initial message for information they already provided. Users often front-load answers — "I want to build a TypeScript framework inspired by Angular with a Rust compiler" already tells you the What (framework), Form (framework/monorepo), and partial Stack (TypeScript, Rust). Don't waste their time re-asking what they already told you.

**Extraction step:** Map the user's initial message against the 7 categories below. For each one, determine:
- **Already answered** — acknowledge what you understood and move on.
- **Partially answered** — acknowledge and ask a targeted follow-up to fill the gap.
- **Not mentioned** — ask the question.

Then interview the user on whatever remains. ONE question at a time. Do NOT dump a questionnaire. Wait for each answer before asking the next.

**The 7 categories (ask only what's missing):**

1. **What** — "What does this project do? What problem does it solve?"
2. **Why** — "Why does this need to exist? What's wrong with existing solutions?"
3. **Who** — "Who uses it? Developers? End users? Both?"
4. **Form** — "What form does it take? Library, CLI, framework, app, API, plugin?"
5. **Principles** — "What are the non-negotiable technical principles?"
6. **Stack** — "What's the tech stack? Languages, runtimes, tools, versions."
7. **Interdits** — "What is explicitly forbidden or out of scope?"

**Pushback is mandatory.** If the user gives vague answers, do NOT advance to the next question. Block the interview until the current answer is concrete and verifiable:

- "What do you mean by 'performant'? Zero runtime overhead? Sub-100ms response? Faster than X?"
- "You said 'simple DX' — simple compared to what? Give me a concrete example."
- "'Modern tooling' is not actionable. Which tools, which versions, why?"
- "'No bad patterns' is a tautology. Name the specific patterns that are forbidden."

Do NOT proceed to Phase 2 with vague principles. Vague constitution = vague specs = vague code.

### Phase 2: Draft

When you have enough information, generate the constitution using the template below. Present it to the user for review. Do NOT write it to disk yet.

### Phase 3: Validate

Ask the user to review:

- "Is anything missing?"
- "Is anything wrong?"
- "Are the interdits complete? What else should never happen?"

Iterate until the user confirms.

### Phase 4: Write

Write the final constitution to `.knowledge/constitution.md`. Create the `.knowledge/` directory if it doesn't exist. If this is an amendment (Phase 0 detected an existing file), add an entry to the Changelog section at the bottom.

## Template

```markdown
# Constitution — <Project Name>

> <One-line description of what this project is>

---

## Identity

**What:** <What the project does — 2-3 sentences max>
**Why:** <Why it exists — what problem it solves that nothing else does>
**Who:** <Target users>
**Form:** <Library | CLI | Framework | App | API | Plugin | Monorepo | etc.>

---

## Principles

> Non-negotiable technical principles. These are **decision filters** — they resolve architecture choices. When two valid approaches exist, principles tell you which one to pick. Aim for 3-5 principles. Fewer than 3 suggests the project identity isn't clear enough. More than 7 suggests some belong in Quality Standards instead.

### <Principle 1 name>

<Concrete, measurable description. Not "fast" — "zero runtime overhead for X". Not "simple" — "no configuration required for basic usage".>

### <Principle 2 name>

<...>

---

## Stack

| Tool   | Version   | Role                           |
| ------ | --------- | ------------------------------ |
| <tool> | <version> | <what it does in this project> |

---

## Interdits

> What this project will NEVER do. Violations are blocking errors.

- <Interdit 1 — concrete and specific>
- <Interdit 2>

---

## Non-goals

> What this project does NOT aim to do. Different from interdits: non-goals are legitimate features we choose not to pursue, not technical violations.

- <Non-goal 1>
- <Non-goal 2>

---

## Quality standards

> Code-level rules that govern **implementation style**. Unlike principles (which resolve architecture choices), quality standards resolve how code is written — linting, testing, error handling, naming. A principle says "what to build"; a quality standard says "how to write it."

- <Standard 1 — e.g. "TypeScript strict, no any">
- <Standard 2 — e.g. "Every public API has tests">
- <Standard 3 — e.g. "Errors at compile-time, not runtime">

---

## Versioning

<Semver | CalVer | other — and the rules around breaking changes, deprecation>

---

## Changelog

> Amendment history. Each entry records what changed, why, and when.

- <YYYY-MM-DD> — Initial constitution ratified.
```

## Rules

- The constitution is the ONLY file this skill produces. It does not produce architecture.md, conventions.md, or any spec.
- Every principle must be **concrete and verifiable**. "Good DX" is not a principle. "No configuration required for basic usage" is.
- Every interdit must be **specific**. "No bad patterns" is not an interdit. "No CSS-in-JS runtime" is.
- The stack table must include **version constraints**, not just tool names.
- If the user's answers reveal scope that spans multiple packages/modules, note it in the Identity section but do NOT define the package structure here — that's `architecture.md`'s job.
- **Principles vs Quality Standards:** If a statement resolves an architecture decision (e.g., "zero runtime overhead" → choose compile-time DI over runtime DI), it's a principle. If it resolves a code style decision (e.g., "no `any` in TypeScript" → how to write type annotations), it's a quality standard. When in doubt, ask: "Does this constrain *what we build* or *how we write it*?"
