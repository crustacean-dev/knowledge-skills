---
name: bootstrap
description: Reverse-engineer an existing codebase into .knowledge/ files (constitution.md, architecture.md, conventions.md, feature specs) by analyzing code, config, tests, and git history. This is the entry point for brownfield projects — existing repos that have code but no .knowledge/ directory yet. It produces the same artifacts that /constitution + /plan + /spec produce for greenfield projects, but derived from what already exists instead of from an interview. ALWAYS use this skill when the user says "bootstrap", "bootstrap this project", "set up knowledge files", "generate knowledge files from code", "analyze this codebase", "create specs from existing code", "reverse engineer this repo", "document this project", "I already have code and need knowledge files", or any request to derive .knowledge/ artifacts from an existing codebase. Also trigger when a project has source code but no .knowledge/ directory and the user wants to start using the plugin — this is the brownfield onboarding path. Do NOT use this skill for greenfield projects (no code yet → use /constitution), for amending existing .knowledge/ files (→ /constitution, /plan, /spec), for writing new feature specs from scratch (→ /spec), for planning architecture (→ /plan), for implementation, task breakdown, code review, or debugging. This skill reads code and produces .knowledge/ files ONLY — it never modifies existing source code.
---

# Bootstrap

You reverse-engineer an existing codebase into `.knowledge/` files. Your job is to analyze what exists and produce the same artifacts that `/constitution`, `/plan`, and `/spec` would produce — but derived from code instead of conversation.

This is a one-time setup skill. After bootstrap, the user continues with the normal plugin flow.

## Phase 0: Detection

Check if `.knowledge/` already exists.

- **If it exists and has content:** STOP. Tell the user that knowledge files already exist. Point them to the amendment skills instead:
  - `/constitution` to amend principles, stack, or interdits
  - `/plan` to amend architecture or conventions
  - `/spec <feature>` to amend a feature spec
  - If they explicitly want to wipe and regenerate, confirm before proceeding — this is destructive.
- **If it doesn't exist or is empty:** Proceed to Phase 1.

## Phase 1: Codebase Analysis

Scan the project systematically. Do NOT read every file — be strategic. Config files, package manifests, a few representative source files, and tests tell you 90% of what you need.

### Step 1: Project shape

Read these files first (if they exist):

- **Package manifests** — `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `*.csproj`, `Gemfile` → language, dependencies, scripts, version constraints
- **Documentation** — `README.md`, `CONTRIBUTING.md`, `docs/` → stated principles, project identity (but verify against code — docs drift)
- **Ignore files** — `.gitignore`, `.dockerignore` → reveals tools and build artifacts
- **Config files** — `tsconfig.json`, `eslint.config.*`, `.prettierrc`, `vite.config.*`, `webpack.config.*`, `Makefile`, `Dockerfile`, `docker-compose.yml`, `rustfmt.toml`, `setup.cfg`, `.editorconfig`
- **CI files** — `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/`
- **Monorepo markers** — `nx.json`, `lerna.json`, `pnpm-workspace.yaml`, `turbo.json`, `Cargo.toml` with `[workspace]`

### Step 2: Directory structure

List the top-level directory tree (2-3 levels deep). Identify:

- Source directories and their purpose
- Test directories and their structure
- Build output directories
- Configuration directories

### Step 3: Module/package inventory

For monorepos: list each package/module with its role and dependencies.
For single projects: list the main directories/modules and their responsibilities.

### Step 4: Code patterns

Sample 3-5 representative source files to identify:

- Naming conventions (files, functions, types, constants)
- Import patterns
- Error handling approach
- Testing patterns (framework, file naming, mocking)
- Code style (formatting, indentation, semicolons)

Pick files that are typical, not exceptional — avoid generated code, vendored deps, or one-off scripts.

### Step 5: Git history

Run `git log --oneline -20` and `git log --format="%s" -50` to determine:

- Commit message format (conventional commits, free-form, ticket prefixes)
- Commit scopes (if any)
- Release patterns (tags, version bumps)

If the repo has few or no commits, skip this step.

### Step 6: Feature detection

Identify the main features/domains of the project by looking at:

- Directory structure (each top-level dir often maps to a feature)
- Package boundaries (in monorepos)
- Route definitions, controller files, module declarations
- README sections that describe functionality

## Phase 2: Interview

Present a summary of what you found. Then ask targeted questions about what you couldn't determine from code alone.

Ask ONE question at a time. Wait for the answer before asking the next. Skip questions you already answered from code analysis.

**Questions (ask only what you couldn't determine):**

1. **Identity** — "Here's what I see: [summary]. Is this correct? What's the one-line description of this project?"
2. **Why** — "Why does this project exist? What problem does it solve? (This is rarely in the code)"
3. **Principles** — "I see these patterns in the code: [list]. Are any of these intentional non-negotiable principles? Or are they just conventions that evolved?"
4. **Interdits** — "Are there things this project should NEVER do that aren't obvious from the code?"
5. **Missing context** — "I see [pattern] but I'm not sure why. Can you explain the reasoning behind [specific decision]?"

**Pushback is mandatory.** If the user gives vague answers, do NOT advance. Block until the answer is concrete:

- "What do you mean by 'performant'? Zero runtime overhead? Sub-100ms response? Faster than X?"
- "'Keep it simple' is not a principle — it doesn't resolve decisions. Simple compared to what? What complexity is forbidden?"
- "'Standard stuff' — there is no standard. What specifically is forbidden? What patterns are required?"

## Phase 3: Generate

Produce the knowledge files in this order. Each file depends on the previous ones.

### 1. constitution.md

Derive from:

- README, CONTRIBUTING.md for stated principles
- Code patterns for implicit quality standards
- User's answers for principles and interdits
- Package manifest `engines`/`volta`/`python_requires` for version constraints

**Inferred principles require confirmation.** When you derive a principle from code patterns rather than user statements, flag it explicitly in the output:

```markdown
### <Principle Name> <!-- BOOTSTRAP: inferred from code — confirm with user -->

<Description. Explain what pattern you observed that led to this inference.>
```

The user must confirm inferred principles before they become non-negotiable. Until confirmed, they are observations, not rules.

**Changelog:** Set the first entry to:
```
- <YYYY-MM-DD> — Initial constitution (bootstrapped from existing codebase).
```

Use this template:

````markdown
# Constitution — <Project Name>

> <One-line description of what this project is>

---

## Identity

**What:** <What the project does — 2-3 sentences max>
**Why:** <Why it exists — what problem it solves>
**Who:** <Target users>
**Form:** <Library | CLI | Framework | App | API | Plugin | Monorepo | etc.>

---

## Principles

> Non-negotiable technical principles. Decision filters — when two valid approaches exist, principles tell you which one to pick. Aim for 3-5.

### <Principle name>

<Concrete, measurable description. Not "fast" — "zero runtime overhead for X".>

---

## Stack

| Tool   | Version   | Role                           |
| ------ | --------- | ------------------------------ |
| <tool> | <version> | <what it does in this project> |

---

## Interdits

> What this project will NEVER do. Violations are blocking errors.

- <Interdit — concrete and specific>

---

## Non-goals

> What this project does NOT aim to do. Different from interdits: non-goals are legitimate features we choose not to pursue.

- <Non-goal>

---

## Quality standards

> Code-level rules that govern implementation style. Unlike principles (which resolve architecture choices), quality standards resolve how code is written.

- <Standard — e.g. "TypeScript strict, no any">

---

## Versioning

<Semver | CalVer | other — and the rules>

---

## Changelog

- <YYYY-MM-DD> — Initial constitution (bootstrapped from existing codebase).
````

### 2. architecture.md

Derive from:

- Directory structure → Structure section
- Package/module inventory → Packages/Modules table
- Config files → Build & Dev table
- CI files → high-level CI/CD info
- Observed patterns → Decisions section (with rationale inferred from code)

For Decisions: every significant choice needs a **Rejected** alternative. If you can't determine what was rejected, mark it:

```markdown
**Rejected:** <!-- BOOTSTRAP: verify with user — couldn't determine alternatives considered -->
```

Use this template:

````markdown
# Architecture — <Project Name>

> Technical structure and decisions for the project.

---

## Overview

<2-3 sentences: what this project is technically, how it's organized.>

---

## Structure

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

### Decision: <title>

**Choice:** <what we do>
**Rationale:** <why — tied to a constitution principle when possible>
**Rejected:** <what we don't do and why>

---

## Diagrams

<Optional: dependency graph, data flow, or system diagram in mermaid or text.>
````

### 3. conventions.md

Derive from:

- Sampled code → Naming conventions, file structure, imports
- Linter/formatter config → Code style rules
- Test files → Testing strategy, test naming, mocking approach
- Git history (from Step 5) → Commit format and scopes
- CI pipeline → Release process

Use this template:

````markdown
# Conventions — <Project Name>

> Code-level rules for implementation. Derived from the constitution's quality standards.

---

## Code style

### Naming

| Element   | Convention   | Example   |
| --------- | ------------ | --------- |
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

<What runs in CI, what triggers what. Keep it high-level.>
````

### 4. Feature specs (optional)

For each identified feature/domain, create `.knowledge/features/<feature>/spec.md` with what you can determine from code.

Only generate specs for **major features** — don't create a spec for every utility file. A good rule: if it has its own directory, route, or package boundary, it might deserve a spec. If it's a helper or utility, it doesn't.

**Every spec from bootstrap is a DRAFT.** Mark the entire file:

```markdown
<!-- BOOTSTRAP DRAFT: This spec was reverse-engineered from code. Run /clarify before using for new development. -->
```

Mark uncertain sections with `<!-- BOOTSTRAP: verify with user -->`. Use Questions ouvertes for behavior you can't determine from code alone.

Use code examples in the project's actual language (not TypeScript if the project is Python/Rust/Go/etc.).

Use this template:

````markdown
<!-- BOOTSTRAP DRAFT: This spec was reverse-engineered from code. Run /clarify before using for new development. -->

# Spec — <Feature Name>

> Package: `<package name if applicable>`
> Feature: <short identifier>

---

## Objective

<What this feature does and why it matters.>

---

## Constraints

> From constitution.md — which principles and interdits apply.

- <Constraint — reference by name>

---

## User stories

### US-1: <Title>

> As a <role>, I want <action> so that <benefit>.

**Input:**

```<language>
// Exact input — types, structure, example values
<code>
```

**Output:**

```<language>
// Exact output
<code>
```

**Acceptance criteria:**

- <Criterion — verifiable>

---

## Edge cases

### EC-1: <Description>

**Input:** <trigger>
**Expected behavior:** <outcome>

---

## Hors scope

- <What this feature won't do>

---

## Questions ouvertes

- <Unresolved points — things bootstrap couldn't determine from code>
````

## Phase 4: Review & Write

Present ALL generated files to the user for review BEFORE writing anything. This is a lot of content — present it file by file:

1. "Here's the constitution I derived. Review it — especially the inferred principles."
2. "Here's the architecture. Review it."
3. "Here's conventions. Review it."
4. "Here are the feature specs I identified: [list]. Want me to show each one?"

After the user confirms each file (with amendments), write them to `.knowledge/`.

## Phase 5: Next Steps

After writing all files, tell the user what to do next:

1. **Review inferred principles** — Any principle marked `<!-- BOOTSTRAP: inferred from code -->` is an observation, not yet a rule. Confirm or remove each one.
2. **Run `/clarify` on each feature spec** — Bootstrap specs are drafts. `/clarify` will find gaps, missing edge cases, and vague criteria.
3. **Use `/spec` for new features** — Bootstrap captures what exists. New features need fresh specs.
4. **Use `/plan <feature>` before implementing** — Bootstrap didn't produce feature-level plans (only project architecture). Each feature needs a plan before `/tasks` and `/implement`.

## Rules

- This skill produces ONLY `.knowledge/` files. It does not modify existing code.
- Do NOT fabricate what you can't determine from code. If you're unsure about a principle, ask. If you can't determine a feature's edge cases from code, put it in Questions ouvertes.
- Mark bootstrap-inferred content with `<!-- BOOTSTRAP: verify with user -->` comments when confidence is low.
- Bootstrap output must be **structurally identical** to output from `/constitution` + `/plan` + `/spec`. Use the templates above — they match the sibling skills exactly.
- Do NOT read every file. Be strategic — config files, package manifests, a few representative source files, and tests tell you 90% of what you need.
- Feature specs from bootstrap are DRAFTS. They capture what exists, not what should exist.
- **Code is the source of truth.** If documentation says one thing and code does another, trust the code. Flag the discrepancy: "README says X but code does Y — I'm going with Y. Is the README outdated, or is the code wrong?"
- Constitution principles derived from code are INFERRED, not stated. Flag them explicitly. The user must confirm before they become non-negotiable.
- Use the project's actual language for code examples in specs. If the project is Python, write Python examples. If Rust, write Rust. Don't default to TypeScript.
