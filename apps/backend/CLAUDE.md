# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
composer setup    # First-time setup: install deps, .env, key:generate, migrate, npm install & build
composer dev      # Start all dev services concurrently: server + queue listener + pail logs + npm dev
composer test     # php artisan config:clear && php artisan test
```

Single test:
```bash
php artisan test --filter TestClassName
./vendor/bin/phpunit tests/Feature/SomeTest.php
```

Code quality:
```bash
composer lint          # Fix formatting with Pint (PSR-12)
composer lint:check    # Check formatting without fixing
composer analyse       # PHPStan static analysis (level 5)
```

API docs (Scramble):
```bash
php artisan scramble:export   # Export OpenAPI 3.1 spec
# Docs UI: http://localhost:8000/docs/api
```

Database:
```bash
php artisan migrate
php artisan tinker
```

## Architecture — DDD

The backend follows **Domain-Driven Design**. Each feature belongs to a bounded context.

```
app/
├── {Context}/                        # Bounded context (e.g. Billing, Catalog, Auth)
│   ├── Domain/
│   │   ├── Models/                   # Eloquent entities
│   │   ├── ValueObjects/             # Immutable typed wrappers
│   │   ├── Events/                   # Domain events (past tense: InvoicePaid)
│   │   ├── Exceptions/               # Domain-specific exceptions
│   │   └── Repositories/            # Repository interfaces (contracts)
│   ├── Application/
│   │   ├── Commands/                 # Command DTOs (write intent)
│   │   ├── Handlers/                 # One handler = one use case
│   │   └── Queries/                  # Read-side queries
│   └── Infrastructure/
│       └── Repositories/            # Eloquent implementations of domain repos
├── Shared/                           # Cross-context base classes and value objects
└── Presentation/
    ├── Api/
    │   ├── Controllers/
    │   ├── Requests/                  # Form Requests (validation)
    │   └── Resources/                 # API Resources (output shaping)
    └── Admin/
        └── Controllers/              # Inertia controllers
```

**Dependency rule**: `Presentation → Application → Domain ← Infrastructure`

Domain has no dependency on Laravel HTTP or Eloquent infrastructure. Handlers are the only entry point from Presentation into the domain.

## Test environment (auto-configured via `phpunit.xml`)

- `DB_DATABASE=:memory:` — in-memory SQLite
- `CACHE_STORE=array`, `SESSION_DRIVER=array`, `QUEUE_CONNECTION=sync`
- `BCRYPT_ROUNDS=4`

Tests: `tests/Feature/Api/` for HTTP-level, `tests/Unit/{Context}/` for domain/handler unit tests.
