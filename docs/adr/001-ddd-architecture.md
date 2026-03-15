# ADR-001: Domain-Driven Design for the Laravel backend

**Status**: `accepted`
**Date**: 2026-03-14
**Deciders**: Adel

## Context

The backend needs a structure that scales as bounded contexts multiply and business logic grows. A flat MVC structure (models/controllers/services) leads to fat controllers, anemic models, and unclear boundaries between domains as the application grows.

## Decision

The Laravel backend (`apps/backend/`) adopts DDD with four layers per bounded context:

```
app/{Context}/
├── Domain/          # Entities, Value Objects, Events, Repository interfaces
├── Application/     # Commands, Handlers (use cases), Queries
├── Infrastructure/  # Eloquent repository implementations
└── (Presentation — shared across contexts in app/Presentation/)
```

Dependency rule: `Presentation → Application → Domain ← Infrastructure`

## Consequences

### Positive
- Business logic is isolated in handlers — testable without HTTP or DB
- Repository interfaces allow fake implementations for fast unit tests
- Bounded contexts have clear boundaries — no silent cross-context coupling
- New use cases = new handler class, existing code is not modified

### Negative
- More files per feature than flat MVC
- Steeper onboarding curve for developers unfamiliar with DDD
- Artisan generators need manual path adjustment (make:model, make:controller)

### Risks
- **Risk**: Developers shortcut the layers under time pressure
- **Mitigation**: Rules enforced in `.claude/rules/laravel-conventions.md` and checked in every `/review`

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| Flat MVC (models/controllers/services) | Does not scale — business logic ends up scattered |
| Hexagonal architecture (ports & adapters) | More ceremony than needed at current stage |
| CQRS with event sourcing | Premature complexity — not required now |
