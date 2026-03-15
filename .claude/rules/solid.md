---
name: solid
description: SOLID principles applied to the DDD + Laravel stack. Enforced in every class written in apps/backend/.
---

SOLID principles are applied within the DDD layer structure. Each principle maps to concrete patterns already in the project.

---

## S — Single Responsibility

One class, one reason to change.

- **Handler**: one public `handle()` method, one use case
- **Repository**: one implementation per aggregate root
- **Form Request**: validates one operation (store ≠ update)
- **API Resource**: shapes output for one entity
- **Value Object**: encapsulates one domain concept

```php
// correct — CreateInvoiceHandler does one thing
final class CreateInvoiceHandler
{
    public function handle(CreateInvoiceCommand $command): Invoice { ... }
}

// forbidden — handler doing multiple unrelated things
final class InvoiceHandler
{
    public function create(...): Invoice { ... }
    public function sendEmail(...): void { ... }
    public function generatePdf(...): string { ... }
}
```

---

## O — Open/Closed

Open for extension, closed for modification.

- Add behavior by creating **new handlers**, not modifying existing ones
- Add new repository methods by **extending the interface** and implementation
- Add new API behaviors with **new Form Requests and Resources**, not by adding conditionals to existing ones

```php
// correct — new use case = new handler
final class UpdateInvoiceHandler { ... }
final class CancelInvoiceHandler { ... }

// forbidden — adding a flag to change existing handler behavior
final class CreateInvoiceHandler
{
    public function handle(CreateInvoiceCommand $command, bool $sendEmail = false): Invoice
    {
        // branching on a flag violates O
    }
}
```

---

## L — Liskov Substitution

Subtypes must be substitutable for their base types.

- **Fake repositories** (`tests/Fakes/`) must honor the full contract of the domain interface — same return types, same exceptions
- **Eloquent repositories** must never return more or less than what the interface promises
- A handler must behave identically whether it receives a real or fake repository

```php
// Both must behave identically from the handler's perspective
final class EloquentInvoiceRepository implements InvoiceRepositoryInterface { ... }
final class InMemoryInvoiceRepository implements InvoiceRepositoryInterface { ... }
```

If a fake silently swallows exceptions that the real implementation throws, tests pass but production fails — LSP violation.

---

## I — Interface Segregation

Clients should not depend on methods they do not use.

- Repository interfaces expose **only the methods actually needed** by the handlers that use them
- Do not create a god `RepositoryInterface` with every possible query method
- Split if two handlers use completely different subsets of a repository

```php
// correct — focused interface
interface InvoiceRepositoryInterface
{
    public function save(Invoice $invoice): void;
    public function findOrFail(int $id): Invoice;
    public function findByUser(int $userId): Collection;
}

// forbidden — god interface
interface InvoiceRepositoryInterface
{
    public function save(Invoice $invoice): void;
    public function findOrFail(int $id): Invoice;
    public function findByUser(int $userId): Collection;
    public function findOverdue(): Collection;
    public function findByStatus(string $status): Collection;
    public function findByDateRange(Carbon $from, Carbon $to): Collection;
    public function countByUser(int $userId): int;
    public function sumByUser(int $userId): int;
    // ... every query the entire app will ever need
}
```

When queries multiply, use dedicated Query classes (read side) rather than bloating the repository interface.

---

## D — Dependency Inversion

High-level modules must not depend on low-level modules. Both must depend on abstractions.

- **Handlers** depend on repository **interfaces** — never on Eloquent classes directly
- **Controllers** depend on **handlers** — never on Eloquent models or repositories
- Bindings are registered in `AppServiceProvider` — this is the only place that knows the concrete implementation

```php
// correct — handler depends on interface
final class CreateInvoiceHandler
{
    public function __construct(
        private InvoiceRepositoryInterface $invoices, // interface, not EloquentInvoiceRepository
    ) {}
}

// forbidden — handler depends on Eloquent implementation directly
final class CreateInvoiceHandler
{
    public function __construct(
        private EloquentInvoiceRepository $invoices, // concrete class
    ) {}
}
```

```php
// AppServiceProvider — the only place that knows the concrete binding
$this->app->bind(InvoiceRepositoryInterface::class, EloquentInvoiceRepository::class);
```

---

## Quick reference

| Principle | In this project means... |
|---|---|
| **S** | One handler = one use case. One repository = one aggregate. |
| **O** | New behavior → new class. Never add flags to existing handlers. |
| **L** | Fake repos must be fully substitutable for real ones in tests. |
| **I** | Repository interfaces expose only what their handlers need. |
| **D** | Handlers inject interfaces. Controllers inject handlers. Only `AppServiceProvider` knows the concrete class. |
