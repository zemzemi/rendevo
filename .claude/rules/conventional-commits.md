---
name: conventional-commits
description: Enforce Conventional Commits format for all git commits in this repository.
---

All commits must follow the Conventional Commits specification.

## Format

```
type(scope): short description

[optional body]

[optional footer]
```

## Types

| Type | When to use |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or updating tests |
| `chore` | Build process, dependencies, config (no production code change) |
| `docs` | Documentation only |
| `style` | Formatting, missing semicolons, etc. (no logic change) |
| `perf` | Performance improvement |

## Scopes for this project

Use a **bounded context** as scope when the change targets a specific domain:

- `billing` — Billing bounded context (invoices, payments, subscriptions)
- `auth` — Authentication & authorization
- `catalog` — Product/service catalog
- `admin` — Inertia admin panel (Presentation/Admin layer)
- `api` — API Presentation layer (controllers, requests, resources)
- `frontend` — Nuxt frontend (`apps/frontend/`)
- `db` — Migrations, seeders
- `queue` — Jobs, events, listeners
- `config` — Configuration files
- `types` — Shared TypeScript types (`packages/shared-types/`)
- `shared` — Cross-context shared domain concepts (`app/Shared/`)
- `deps` — Dependency updates (`composer.json`, `package.json`)

## Examples

```
feat(billing): add invoice creation handler and API endpoint
fix(auth): prevent session fixation on login
refactor(admin): extract InvoiceTable into reusable component
test(billing): add handler unit tests for CreateInvoiceHandler
feat(api)!: rename /api/users to /api/accounts
chore(deps): update laravel/framework to 12.x
feat(frontend): add invoice list page with useFetch composable
```

## Rules

- Subject line max 72 characters
- Use imperative mood: "add" not "added", "fix" not "fixed"
- No period at the end of the subject line
- Breaking changes: add `!` after scope — `feat(api)!: rename endpoint`
