---
name: review
description: Run a structured code review on recently modified files. Checks DDD layer boundaries, Laravel conventions, Vue/TypeScript patterns, and security basics.
argument-hint: "[path/to/file or directory] (optional — defaults to current changes)"
---

The user wants a code review. Target: $ARGUMENTS

## Changes to review

!`git diff HEAD --stat 2>/dev/null | head -30 || echo "Run from a git repository"`

---

## Step 1 — Identify files to review

If $ARGUMENTS specifies a file or directory, review that target specifically.

Otherwise, review all files modified compared to HEAD shown above.

---

## Step 2 — Read each file in full before commenting

Do not comment based on partial context. Read the complete file, understand its role in the system, then assess.

---

## Step 3 — Apply the review checklist

### PHP / Laravel — DDD

- [ ] DDD dependency rule respected? (`Presentation → Application → Domain ← Infrastructure`)
- [ ] Controller is thin — builds a Command, calls a Handler, returns a Resource? (max 5 lines per method)
- [ ] No business logic in controller (no direct Eloquent calls except simple reads)?
- [ ] Handler used — not a service class or inline logic?
- [ ] Command DTO used to pass data from Presentation to Application layer?
- [ ] Domain model does not import from `Presentation/` or `Infrastructure/`?
- [ ] Repository interface defined in `Domain/`, Eloquent implementation in `Infrastructure/`?
- [ ] Validation in Form Request, not in controller?
- [ ] Authorization via Policy, not inline role checks?
- [ ] `$fillable` explicit (no `$guarded = []`)?
- [ ] No N+1 queries (relationships eager-loaded with `with()`)?
- [ ] API response uses a Resource class — no raw arrays or `$model->toArray()`?
- [ ] No sensitive fields exposed in JSON (password, remember_token, etc.)?
- [ ] New routes have appropriate middleware?
- [ ] SOLID principles respected? (see `rules/solid.md`)
- [ ] `./vendor/bin/pint` formatting applied?

### Tests

- [ ] Feature test covers: happy path, 401, 403, 422?
- [ ] Unit tests for handlers use a fake in-memory repository (`tests/Fakes/`)?
- [ ] Factories used for all test data — no raw `DB::insert()`?
- [ ] Assertions use `assertJsonPath()` over `assertJson()` with full arrays?

### Vue / TypeScript (Inertia + Nuxt)

- [ ] Props fully typed with `defineProps<{}>()`?
- [ ] No `any` — use `unknown` and narrow?
- [ ] Composables are SSR-safe (no `window`/`document` at module level)?
- [ ] Inertia: using `useForm()` for forms, `<Link>` for navigation?
- [ ] Nuxt: using `useFetch`/`useAsyncData` instead of raw `fetch` in `<script setup>`?
- [ ] No `v-html` with untrusted/user-generated content?
- [ ] CSS variable tokens used — no hardcoded Tailwind palette colors?

### Security

- [ ] No hardcoded secrets or environment values?
- [ ] User input never reaches a raw SQL query?
- [ ] Error responses do not leak stack traces or internal details?

---

## Step 4 — Format findings

For each issue found:

**[BLOCKING | WARNING | SUGGESTION]** `path/to/file:line`

> Quote the problematic code

Problem: explain why it is an issue.

Fix:
```
// corrected code
```

For security-specific findings, flag them and recommend a full audit with the `security` agent.

---

## Step 5 — Summary

End with:
- Count of blocking / warning / suggestion items
- Overall assessment: ready to merge / needs fixes / significant rework needed
