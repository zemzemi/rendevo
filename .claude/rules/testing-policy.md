---
name: testing-policy
description: Testing requirements for all new backend code. Every feature ships with tests.
---

## Rule: no feature without a Feature test

Every new API endpoint or admin action must have a corresponding Feature test covering:
1. The happy path (authenticated, valid data)
2. Unauthenticated access → 401
3. Forbidden access (wrong role/ownership) → 403
4. Invalid input → 422 with validation errors

## Rule: two test types, two different strategies

| Test type | Location | DB strategy |
|---|---|---|
| Feature (HTTP level) | `tests/Feature/Api/` or `tests/Feature/Admin/` | Real in-memory SQLite — `use RefreshDatabase` |
| Unit (handler/domain level) | `tests/Unit/{Context}/Application/` | Fake in-memory repository — no DB at all |

**Feature tests** hit a real SQLite in-memory database. Never mock Eloquent models or the DB facade in Feature tests.

**Unit tests for handlers** use a fake repository implementing the domain interface — no database, no Eloquent. This is not mocking: it is a real in-memory implementation of the repository contract.

```php
// Feature test — real DB
class StoreInvoiceTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_create_invoice(): void
    {
        $user = User::factory()->create();
        $this->actingAs($user)->postJson('/api/invoices', [...])
            ->assertStatus(201);
    }
}

// Handler unit test — fake repository, no DB
class CreateInvoiceHandlerTest extends TestCase
{
    public function test_handler_creates_invoice_and_fires_event(): void
    {
        Event::fake();
        $repo = new InMemoryInvoiceRepository(); // implements InvoiceRepositoryInterface
        $handler = new CreateInvoiceHandler($repo);

        $invoice = $handler->handle(new CreateInvoiceCommand(userId: 1, amount: new Money(5000), ...));

        $this->assertCount(1, $repo->all());
        Event::assertDispatched(InvoiceCreated::class);
    }
}
```

## Rule: use factories for all Feature test data

Never use raw `DB::insert()` or `Model::create()` with hardcoded data in Feature tests.

```php
// correct
$invoice = Invoice::factory()->for($user)->create(['status' => 'pending']);

// forbidden
DB::insert('insert into invoices ...', [...]);
```

## Rule: assert at the right level

| What to verify | Use |
|---|---|
| HTTP status | `assertStatus(201)` |
| JSON shape | `assertJsonPath('data.status', 'pending')` |
| DB state | `assertDatabaseHas('invoices', ['status' => 'paid'])` |
| DB absence | `assertDatabaseMissing(...)` |
| Soft delete | `assertSoftDeleted(...)` |

Avoid `assertJson()` with full arrays — brittle. Prefer `assertJsonPath()` for specific fields.

## Test file locations

```
tests/
├── Feature/
│   ├── Api/                          # HTTP request → response → DB (Feature tests)
│   └── Admin/                        # Inertia admin panel (Feature tests)
├── Unit/
│   └── {Context}/
│       ├── Domain/                   # Value object and domain model logic
│       └── Application/             # Handler tests using fake repositories
└── Fakes/
    └── {Context}/
        └── InMemory{Resource}Repository.php  # Fake repository implementations
```

## Fake repositories (`tests/Fakes/`)

Fake repositories are real PHP classes implementing the domain repository interface, storing entities in an in-memory array. They are shared across all handler unit tests in the same bounded context.

```php
// tests/Fakes/Billing/InMemoryInvoiceRepository.php
final class InMemoryInvoiceRepository implements InvoiceRepositoryInterface
{
    private array $items = [];

    public function save(Invoice $invoice): void
    {
        $this->items[$invoice->id] = $invoice;
    }

    public function findOrFail(int $id): Invoice
    {
        return $this->items[$id] ?? throw new ModelNotFoundException();
    }

    public function findByUser(int $userId): Collection
    {
        return collect($this->items)->filter(fn($i) => $i->user_id === $userId)->values();
    }

    /** Helper for assertions in tests */
    public function all(): array
    {
        return $this->items;
    }
}
```

## Running tests

```bash
php artisan test                              # Full suite
php artisan test --filter InvoiceTest         # Single class
php artisan test tests/Feature/Api/           # Directory
php artisan test tests/Unit/Billing/          # Bounded context unit tests
```
