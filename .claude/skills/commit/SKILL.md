---
name: commit
description: Generate a conventional commit message based on staged changes. Analyzes the diff and maps files to DDD bounded context scopes.
argument-hint: "[optional context about the intent]"
disable-model-invocation: true
---

The user wants to generate a commit message. Additional context: $ARGUMENTS

## Current changes

Staged changes:
```
!`git diff --staged 2>/dev/null || echo "No staged changes"`
```

Modified files:
```
!`git status --short 2>/dev/null`
```

## Step 1 — Analyze the diff

Read the staged diff above. If nothing is staged, look at unstaged changes. Understand what changed and why — not just what files, but the intent of the change.

## Step 2 — Identify the scope

Map the changed files to a bounded context or project scope:

| Files changed | Scope |
|---|---|
| `app/{Context}/Domain/` | the bounded context name (e.g. `billing`, `auth`, `catalog`) |
| `app/{Context}/Application/` | the bounded context name |
| `app/{Context}/Infrastructure/` | the bounded context name |
| `app/Presentation/Api/` | `api` |
| `app/Presentation/Admin/` | `admin` |
| `app/Shared/` | `shared` |
| `apps/frontend/` | `frontend` |
| `database/migrations/` | `db` |
| `routes/` | `api` or `admin` |
| `packages/shared-types/` | `types` |
| `composer.json` / `package.json` | `deps` |
| `.claude/` | `config` |
| `docs/` | `docs` |

If the change spans multiple scopes, pick the primary one.

## Step 3 — Choose the type

| Type | When |
|---|---|
| `feat` | New behavior exposed to users or consumers |
| `fix` | Bug fix |
| `refactor` | Code restructured without behavior change |
| `test` | Tests added or updated |
| `chore` | Build, config, dependencies |
| `docs` | Documentation only |
| `perf` | Performance improvement |

## Step 4 — Write the message

Format: `type(scope): short description`

Rules:
- Imperative mood: "add" not "added"
- Max 72 characters on the subject line
- No period at the end
- Breaking change: add `!` — `feat(api)!: rename endpoint`

If the change warrants a body (complex refactor, migration, breaking change), add it after a blank line.

## Step 5 — Output

Provide:
1. The commit message ready to copy-paste
2. A one-line explanation of why you chose that type and scope
3. If multiple commits would be more appropriate (e.g. migration + handler are separate concerns), say so and provide one message per commit

Do not run `git commit` — only produce the message.
