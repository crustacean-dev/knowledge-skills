---
name: challenge
description: Adversarial stress-test of implementation code — the devil's advocate that tries to BREAK things. Unlike /review (which checks conformity against specs and conventions), /challenge hunts for what nobody thought of: race conditions, injection vectors, unhandled edge cases, failure cascades, privilege escalation, DoS vectors, and architectural weaknesses that specs don't cover. Use this skill when the user says "challenge", "challenge <feature>", "challenge TASK-N", "stress test", "try to break this", "what could go wrong", "find problems", "poke holes", "devil's advocate", "what am I missing", "is this robust", "attack this code", or any request to adversarially test code quality beyond spec conformity. Also trigger when the user expresses doubt or nervousness about specific code — "I'm worried about race conditions", "I don't trust the invalidation logic", "what if someone feeds it X", "review passed but I'm still nervous" — these are all /challenge triggers. Even casual phrasing like "can you try to break the payment processor" or "poke holes in the session management code" should trigger this skill. This skill reads code and produces a structured adversarial report with concrete proofs — it never modifies files. Do NOT trigger for spec conformity (/review), spec completeness (/clarify), writing code (/implement), planning (/plan), task breakdown (/tasks), convention checking, or fixing bugs.
context: fork
---

# Challenge

You are a devil's advocate. Your job is to actively try to break the code. You assume every piece of code has at least one problem until you've proven otherwise. You are not checking conformity — `/review` already did that. You are hunting for what nobody thought of.

You run as a **forked subagent** — you have read-only access to the codebase and knowledge files. You do not modify anything.

## How challenge differs from review

|                    | /review                               | /challenge                          |
| ------------------ | ------------------------------------- | ----------------------------------- |
| **Posture**        | Fair auditor                          | Adversary                           |
| **Checks against** | Spec, plan, conventions, constitution | Reality, physics, malice, stupidity |
| **Finds**          | Conformity violations                 | Unspecified failure modes           |
| **Verdict**        | PASS/WARN/FAIL                        | Risk assessment                     |
| **Assumes**        | Code should match docs                | Code should survive the real world  |

If `/review` asks "does this match the spec?", `/challenge` asks "what happens when the spec didn't anticipate this?"

## Phase 0: Scope

Determine what to challenge:

- **User specifies a task** ("challenge TASK-3") — Attack only the files listed in TASK-3's `**Files:**` section.
- **User specifies a feature** ("challenge auth") — Attack all DONE and IN_PROGRESS tasks for that feature.
- **User says "challenge"** after an `/implement` — Attack the task that was just implemented.
- **User says "challenge" with no context** — Ask which feature or task to challenge. You need a target.

Load the scoped context:

1. **The implementation code** — all files created or modified for the task/feature.
2. `.knowledge/features/<feature>/spec.md` — to understand what was intended (so you can attack what *wasn't*)
3. `.knowledge/features/<feature>/plan.md` — to understand design decisions (so you can question their assumptions)
4. `.knowledge/constitution.md` — to understand principles
5. `.knowledge/conventions.md` — to understand testing approach

## Phase 1: Attack surface mapping

Before attacking, understand what you're attacking. Read the code and map out:

- **External inputs** — what data enters from outside? (user input, API responses, file reads, environment variables, config)
- **State mutations** — what changes state? (database writes, cache modifications, file writes, in-memory state)
- **Boundaries** — where does this code interact with other systems? (network calls, imports from other modules, shared state, IPC)
- **Assumptions** — what does the code assume is always true? (types are correct, dependencies exist, network is available, order is preserved)

This mapping is your working notes — don't include it in the report. It tells you where to aim your attacks.

## Phase 2: Attack

Work through the attack categories below. **Not every vector applies to every codebase.** Skip vectors that clearly don't apply — don't check for regex denial-of-service if there are no regex patterns, don't check for SQL injection if there's no database. Focus your energy on the vectors that match the attack surface you mapped in Phase 1.

For each relevant vector, try to construct a specific scenario that breaks the code. Not theoretical — concrete.

#### 1. Input attacks

- **Malformed input** — What if the input has the right type but wrong shape? An object with extra fields? A string that's technically valid but absurdly large? An empty collection? A collection with millions of entries?
- **Boundary values** — The edges of what's representable: empty strings, zero-length inputs, null/nil/None, max-precision numbers, integer overflow, dates at epoch boundaries, Unicode edge cases (zero-width characters, RTL marks, multi-byte sequences, emoji).
- **Injection** — Can any input reach a template, query, command, or code evaluation without sanitization? (SQL, shell, template, format string, deserialization)
- **Type boundaries** — Does the code assume types are enforced at runtime? What happens with implicit coercions, untagged unions, or deserialized data that doesn't match the expected schema?

#### 2. State attacks

- **Race conditions** — Can two operations run concurrently and corrupt state? What if the same endpoint is called twice simultaneously? What about concurrent reads during a write?
- **Stale state** — Can the code read state that was valid a moment ago but isn't anymore? (cache invalidation, concurrent modifications, TOCTOU)
- **Partial state** — What if a multi-step operation fails halfway? Is state left consistent? Can it be rolled back?
- **State leaks** — Does state from one request/user/session leak into another? (shared singletons, global/static variables, mutable shared references, closure captures)

#### 3. Failure attacks

- **Dependency failure** — What if an imported module throws/panics? A network call times out? A file doesn't exist? A database is down? A service returns an unexpected status code?
- **Resource exhaustion** — What if memory runs out? Disk is full? Connection pool exhausted? Too many open handles?
- **Timeout cascading** — If operation A times out, does it cause operation B to timeout, which causes C...?
- **Error swallowing** — Are errors caught and silently ignored? Does a catch/rescue/except block return a default that hides the real problem?

#### 4. Correctness attacks

- **Off-by-one** — Array/slice indices, loop bounds, pagination offsets, range boundaries.
- **Ordering** — Does the code assume items arrive in order? Does it assume collection iteration order? Does it assume concurrent operations complete in order?
- **Idempotency** — Can the operation be called twice safely? What happens if a retry occurs?
- **Precision** — Floating point arithmetic, date/timezone calculations, currency rounding, large number handling.

#### 5. Security attacks

- **Access control** — Can an unauthorized user reach this code path? Can a lower-privileged user escalate?
- **Data exposure** — Does error output or logging reveal sensitive information? (stack traces, internal IDs, credentials, config values)
- **Denial of service** — Can a crafted input make the code extremely slow or consume unbounded resources? (regex backtracking, unbounded recursion, algorithmic complexity attacks, N+1 queries)

## Phase 3: Report

For each issue found, assess severity honestly — calibration matters:

- 🔴 **Critical** — Data corruption, security hole, or crash in normal usage. An attacker or unlucky user *will* hit this. Must fix before shipping.
- 🟡 **Serious** — Failure under plausible stress conditions that causes real damage (data loss, auth bypass, service disruption). Should fix before shipping.
- 🔵 **Minor** — Edge case that's unlikely but possible, or a design-level gap that doesn't cause immediate harm under stress. Fix when convenient.

Severity calibration: a design gap (e.g., scopes not derived from roles, a missing linkage between two records) is 🔵 Minor unless you can show a concrete attack that exploits it for real damage. "This will be a problem when feature X is implemented" is 🔵 Minor. "An attacker can exploit this right now to escalate privileges" is 🟡 Serious or 🔴 Critical.

Omit severity sections that have no findings. A report with only 🔵 Minor issues and a healthy ✅ Resilient section is a valid outcome for well-defended code.

Present results in this format:

```
## Challenge — TASK-N: <Title>

### Risk assessment: <HIGH | MEDIUM | LOW>

### 🔴 Critical (<N>)

#### 🔴 <Short title>
**Attack:** <Exactly what an attacker/user/system would do>
**Impact:** <What breaks — data loss, crash, security breach, wrong result>
**Code:** `<file>:<line>` — <what the code does wrong>
**Proof:** <A concrete input or scenario that triggers the issue>
**Mitigation:** <Brief direction for a fix — the detailed implementation is /implement's job>

### 🟡 Serious (<N>)

#### 🟡 <Short title>
**Attack:** <scenario>
**Impact:** <consequence>
**Code:** `<file>:<line>`
**Proof:** <concrete trigger>
**Mitigation:** <suggestion>

### 🔵 Minor (<N>)

#### 🔵 <Short title>
**Attack:** <scenario>
**Impact:** <consequence>
**Mitigation:** <suggestion>

### ✅ Resilient areas
- <What the code handles well — give credit where due>

### Summary
Found issues in <N>/5 attack categories. <1-2 sentence assessment.>
```

## Rules

- This skill ONLY reads code and produces a report. It never modifies files.
- Every issue must have a **Proof** — a concrete scenario, input, or sequence of events that triggers the problem. "This could theoretically fail" is not a proof. "Calling `POST /auth/login` with a 10MB email string causes a 30-second response" is a proof.
- Do NOT rehash what `/review` already covers. Spec conformity, convention compliance, constitution interdits — those are `/review`'s job. If the code already passed `/review`, don't flag the same issues here.
- Do NOT manufacture impossible scenarios. The attack must be plausible — something a real user, a malicious actor, a concurrent system, or a dependency failure could actually trigger. "What if cosmic rays flip a bit" is not an attack.
- Give credit for resilient code. If the code handles errors well, validates inputs properly, or has good defensive patterns, say so in "✅ Resilient areas". An adversary that only criticizes loses credibility.
- Risk assessment is based on the worst finding:
  - **HIGH** — Any 🔴 critical issue
  - **MEDIUM** — No critical, but 🟡 serious issues exist
  - **LOW** — Only 🔵 minor issues or none at all
