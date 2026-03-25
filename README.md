# knowledge-skills

A Claude Code plugin for spec-driven development. 9 skills covering the full lifecycle — from project principles to adversarial code review.

The plugin produces structured `.knowledge/` files that accumulate project context: constitution, architecture, conventions, feature specs, plans, and task breakdowns. Each skill reads what came before it and writes the next artifact, so implementation never has to guess.

## Installation

Add the marketplace, then install the plugin:

```shell
/plugin marketplace add crustacean-dev/knowledge-skills
/plugin install knowledge-skills@crustacean-dev
```

## The workflow

```
                        PROJECT SETUP
                        ─────────────
               /constitution ──▶ /plan (project)
                     │          architecture.md
                     │          conventions.md
                     ▼
              ── PER FEATURE ──────────────────────────
             │                                         │
             │   /spec ◀──────▶ /clarify               │
             │     │          (audit loop)              │
             │     ▼                                    │
             │   /plan <feature>                        │
             │     │                                    │
             │     ▼                                    │
             │   /tasks                                 │
             │     │                                    │
             │     ▼                                    │
             │   /implement ──▶ /review ──▶ /challenge  │
             │                                         │
              ─────────────────────────────────────────
```

**Greenfield** (new project): start at `/constitution`, then `/plan` for project-level architecture, then loop per feature.

**Brownfield** (existing code): start at `/bootstrap` — it produces constitution + architecture + conventions + draft feature specs in one pass. Then continue with `/clarify` on each draft spec and pick up the normal per-feature flow.

## Skills

### Define

| Skill | Command | Produces | Purpose |
|-------|---------|----------|---------|
| **Constitution** | `/constitution` | `.knowledge/constitution.md` | Project identity, non-negotiable principles, tech stack, interdits, quality standards. The governing document — every other skill reads it. |
| **Plan** | `/plan` or `/plan <feature>` | `.knowledge/architecture.md` + `conventions.md` (project) or `features/<feature>/plan.md` + `api/*.d.ts` (feature) | Architecture decisions, directory structure, conventions, or feature-level technical plans with typed contracts. |
| **Spec** | `/spec <feature>` | `.knowledge/features/<feature>/spec.md` | What a feature does — user stories with typed input/output examples, acceptance criteria, edge cases. Behavior only, not implementation. |
| **Clarify** | `/clarify <feature>` | Updates `spec.md` | Adversarial spec reviewer. 10 systematic checks for gaps, vague criteria, missing edge cases, constitution violations. Every FAIL gets a concrete fix. |

### Build

| Skill | Command | Produces | Purpose |
|-------|---------|----------|---------|
| **Tasks** | `/tasks <feature>` | `.knowledge/features/<feature>/tasks.md` | Ordered implementation tasks derived from spec + plan. Each task has acceptance criteria and file targets. |
| **Implement** | `/implement` or `/implement TASK-N` | Code + tests | Writes code for one task at a time. Reads the task's ACs, the plan, architecture, and conventions — then writes exactly what's needed. |

### Verify

| Skill | Command | Produces | Purpose |
|-------|---------|----------|---------|
| **Review** | `/review` or `/review <feature>` | Conformity report | Checks implementation against specs, conventions, and constitution. Structured pass/warn/fail report with file:line references. |
| **Challenge** | `/challenge` or `/challenge <feature>` | Adversarial report | Devil's advocate — tries to break the code. Race conditions, injection vectors, failure cascades, edge cases that specs don't cover. |

### Onboard

| Skill | Command | Produces | Purpose |
|-------|---------|----------|---------|
| **Bootstrap** | `/bootstrap` | All `.knowledge/` files | Reverse-engineers an existing codebase into knowledge files. One-time setup for brownfield projects. Produces the same artifacts as the define skills, but derived from code analysis instead of interviews. |

## What goes in `.knowledge/`

```
.knowledge/
├── constitution.md            # Principles, stack, interdits
├── architecture.md            # Structure, packages, build, decisions
├── conventions.md             # Naming, testing, commits, CI/CD
└── features/
    └── <feature>/
        ├── spec.md            # What to build (behavior)
        ├── plan.md            # How to build it (technical)
        ├── api/*.d.ts         # Typed contracts
        └── tasks.md           # Implementation task list
```

## Key design principles

**Specs describe what, not how.** `/spec` captures behavior and acceptance criteria. `/plan` captures architecture and algorithms. This separation prevents implementation details from leaking into requirements.

**Every decision has a rejected alternative.** Architecture decisions in `/plan` require a `Rejected:` field. If there's no alternative, it's not a decision — it's a fact.

**Pushback is mandatory.** Every interview skill (constitution, plan, spec) blocks on vague answers. "Keep it simple" is not a principle. "No configuration required for basic usage" is.

**Code is the source of truth.** `/bootstrap` trusts code over documentation. When README says one thing and code does another, it flags the discrepancy and goes with the code.

**Inferred vs. stated.** `/bootstrap` marks principles derived from code patterns as inferred — the user must confirm before they become non-negotiable.

## License

MIT
