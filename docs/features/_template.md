# Feature: {Feature Name}

**Bounded context**: {Context}
**Status**: `planned` | `in-progress` | `done`
**Last updated**: {YYYY-MM-DD}

## Summary

One or two sentences describing what this feature does and for whom.

## Endpoints

| Method | URI | Auth | Description |
|---|---|---|---|
| `POST` | `/api/{resources}` | required | Create a {resource} |
| `GET` | `/api/{resources}` | required | List {resources} |
| `GET` | `/api/{resources}/{id}` | required | Get a single {resource} |
| `PUT` | `/api/{resources}/{id}` | required | Update a {resource} |
| `DELETE` | `/api/{resources}/{id}` | required | Delete a {resource} |

## Domain

**Entity**: `app/{Context}/Domain/Models/{Resource}.php`
**Key fields**:
- `field_name` — description, type, constraints

**Value Objects**:
- `{ValueObject}` — what it encapsulates

**Domain events fired**:
- `{ResourceCreated}` — when, what listeners react

## Business rules

- Rule 1: describe the invariant or constraint
- Rule 2: ...

## Application layer

| Handler | Command | Description |
|---|---|---|
| `Create{Resource}Handler` | `Create{Resource}Command` | Creates a new {resource} |

## Known limitations

- Limitation or edge case not yet handled
- Deliberate simplification and why

## Related

- ADR: [link if a decision was made for this feature]
- Related features: [links to other feature docs if coupled]
