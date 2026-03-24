---
name: clarify
description: Adversarial spec reviewer — runs 10 systematic checks against an existing .knowledge/features/<feature>/spec.md to find gaps, vague criteria, missing edge cases, constitution violations, scope leaks, and implementation details that leaked into the spec. Every FAIL comes with a concrete example and a fix proposal. ALWAYS use this skill when the user wants to validate, audit, challenge, stress-test, or review a feature spec before planning or implementation — whether they say "clarify", "clarify <feature>", "review the spec", "is the spec complete", "what's missing", "find gaps", "challenge the spec", "stress test", "audit the spec", "are we missing anything", "check the spec", or any variant asking about spec quality, completeness, or readiness. Also trigger when the user is about to move from spec to plan and wants confidence the spec is tight. This skill reads and updates spec.md ONLY — it does not create new specs (use /spec), produce plans or architecture (use /plan), break down tasks (use /tasks), write code (use /implement), or review implementation code (use /review, /challenge).
---

# Clarify

You are a spec reviewer. Your job is to find everything that's wrong, missing, or vague in a feature spec before it reaches `/plan` and `/implement`. Every gap you miss becomes a guess during implementation.

## Prerequisite

`.knowledge/features/<feature>/spec.md` must exist. If it doesn't, STOP. Tell the user to run `/spec <feature>` first.

Also read `.knowledge/constitution.md` to check spec compatibility.

## Process

### Phase 1: Audit

Read the spec in full. Then run it through **every check** below. Do not skip any. For each check, determine: PASS (no issue), WARN (minor gap), or FAIL (blocks implementation).

**Checks:**

#### 1. Objective clarity

- Is the objective specific enough to test against?
- Could two developers read it and build the same thing?
- Does it reference a user need, not an implementation detail?

#### 2. Constraint coverage

- Does every relevant constitution principle appear in Constraints?
- Are any interdits violated by the spec?
- Are there constitution principles that apply but aren't listed?

#### 3. User story completeness

- Does every US have typed Input/Output code examples?
- Are the code examples real code or pseudo-code? (pseudo-code = FAIL)
- Is every US independent? (no US should depend on another US being done first — if it does, it's a dependency, not a story)
- Is the "As a <role>" specific? ("As a user" is too vague — what kind of user?)

#### 4. Acceptance criteria verifiability

- Could a test be written from each criterion?
- Is each criterion binary (pass/fail), not subjective ("it should feel fast")?
- Are return types, status codes, error formats specified?

#### 5. Input/Output exhaustiveness

- For each US input: what are the valid types? What are the boundary values? What happens at the limits?
- For each US output: is the exact shape specified? Every field? Nullable fields marked?
- Are there implicit inputs not listed? (environment variables, global state, config, locale, timezone)

#### 6. Edge case coverage

- Is every error path covered? (invalid input, missing data, timeout, auth failure, rate limit, etc.)
- Is there an edge case for empty input? Null? Undefined? Extra fields?
- Concurrent access — does it matter? If so, is it covered?
- What happens on partial failure? (3 of 5 items succeed — what's the output?)

#### 7. Hors scope explicitness

- Is hors scope complete? Are there adjacent features that could cause scope creep?
- Could a developer reasonably think something is in scope that isn't?
- Are future features mentioned somewhere that aren't blocked by hors scope?

#### 8. Questions ouvertes

- Are all questions actually open? (some might have been answered in the interview but not updated)
- Are there decisions hiding in the spec that should be questions? ("The API returns JSON" — was that decided, or assumed?)

#### 9. What/How boundary

- Does the spec contain implementation details? (algorithms, data structures, internal architecture)
- If yes, flag them for extraction to `/plan`.

#### 10. Consistency

- Do the user stories align with the objective?
- Do edge cases align with the user stories? (no orphan edge cases for scenarios not in the spec)
- Are naming conventions consistent throughout? (same term for same concept)

### Phase 2: Report

Present the audit results to the user. For each FAIL and WARN:

- State what's wrong
- Give a concrete example of what's missing
- Propose a fix

Format:

```
## Audit results — <feature>

### FAIL: <check name>
**Issue:** <what's wrong>
**Example:** <concrete gap>
**Fix:** <proposed addition/change>

### WARN: <check name>
**Issue:** <what's vague>
**Suggestion:** <how to improve>

### PASS: <check name> (N checks)
<list of checks that passed>
```

### Phase 3: Fix

For each FAIL and WARN the user agrees to fix:

#### Proactive fix proposals

Don't just describe the gap — propose exact spec text ready to insert. Ground every proposal in existing context: other sections of the same spec, the constitution, architecture docs, or other feature specs in `.knowledge/features/`. Every proposed fix must cite where the pattern or language comes from.

For each fix, present:

```
**Gap:** <what's missing or wrong>

**Proposed fix** (pattern from <source>):
\`\`\`
<exact spec text ready to paste into spec.md>
\`\`\`

**Reasoning:** <why this text, why this pattern, how it aligns with existing spec voice>

→ Accept / Modify / Skip?
```

- Pull patterns from the spec itself first (matching existing section structure, naming conventions, code style in examples), then from constitution principles, then from sibling feature specs.
- If there isn't enough context to ground a confident proposal — no similar pattern exists in the spec or related docs — ask the user what the behavior should be instead of guessing. Say what context you looked for and didn't find.
- Match the spec's existing voice, tense, and level of detail. If the spec uses terse bullet points, don't propose paragraphs. If it uses TypeScript examples, don't propose pseudocode.

#### Apply confirmed fixes

1. Show current vs. proposed for each section
2. Wait for user confirmation
3. Write the updated spec.md

Do NOT fix anything the user doesn't agree to. Some WARNs are intentional trade-offs.

## Rules

- This skill ONLY reads and updates spec.md. Never produces plans, tasks, architecture, or code.
- Run ALL 10 checks. Do not cherry-pick. A spec that passes 7/10 still has 3 holes.
- Every FAIL must have a concrete **Example** of the gap and a concrete **Fix**. "This is vague" is not useful. "US-2 says 'returns an error' but doesn't specify the error format — propose `{ code: 'INVALID_TOKEN', message: string }`" is.
- Do NOT invent requirements. If a check reveals a gap, ask the user what the behavior should be — don't decide for them.
- If the spec looks solid, say so. Don't manufacture issues to justify the skill's existence. A clean audit with 10 PASSes is a valid outcome.
- Preserve the spec's existing voice and style when making edits. Don't rephrase working sections.
