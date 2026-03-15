---
name: bug
description: Structured bug investigation and fix — reproduce, identify root cause in the correct DDD layer, fix minimally, add a failing test.
argument-hint: "[description, error message, or file path]"
---

The user is reporting a bug. Description: $ARGUMENTS

Follow these steps in order. Do not skip to a fix without completing the investigation.

---

## Step 1 — Understand the bug

Extract from $ARGUMENTS:
- **Symptom**: what the user sees (error message, wrong output, unexpected behavior)
- **Expected**: what should happen instead
- **Context**: which layer (API, handler, domain, frontend, admin), which feature, which route or action

If any of these is unclear, ask before proceeding.

---

## Step 2 — Reproduce the bug

Read all relevant files:
- The route, controller, and form request
- The handler and command
- The domain model and repository
- The test file if one exists

Trace the full execution path from the HTTP request to the response or side effect. Identify exactly where the expected behavior diverges from the actual behavior.

Do not guess. If you cannot trace the cause with certainty, say so and ask for additional context (logs, stack trace, input data).

---

## Step 3 — Identify the root cause

Classify the bug:

| Category | Examples |
|---|---|
| **Validation** | Missing rule, wrong type, incorrect constraint |
| **Authorization** | Policy method wrong, middleware missing, ownership check incorrect |
| **Domain logic** | Invariant not enforced, wrong state transition, event not fired |
| **Data** | N+1 query, missing eager load, wrong column, nullable not handled |
| **Infrastructure** | Repository implementation wrong, binding missing in `AppServiceProvider` |
| **Presentation** | Resource exposes wrong fields, wrong HTTP status, missing error handling |
| **Frontend** | Wrong API endpoint, unhandled error state, reactivity issue |
| **DDD violation** | Logic in controller, direct DB call bypassing domain, cross-context import |
| **SOLID violation** | SRP broken (handler doing too much), DIP broken (depends on concrete class) |

State the root cause clearly before writing any fix.

---

## Step 4 — Fix

Apply the minimal fix that resolves the root cause. Do not refactor surrounding code unless it is directly part of the problem.

If the bug is in the wrong layer (e.g. business logic in a controller), fix it in the correct layer — do not patch it where it is.

---

## Step 5 — Add or update a test

Every bug fix must be accompanied by a test that:
- Fails before the fix
- Passes after the fix

If a Feature test already covers the scenario, explain why it did not catch the bug and update it. If no test exists, write one following `/test` conventions.

---

## Step 6 — Output

1. **Root cause**: one clear sentence
2. **Files changed**: list with paths
3. **Fix**: the corrected code
4. **Test**: the test that catches this bug
5. **Prevention note**: which convention or rule, if enforced, would have prevented this bug
