---
name: laravel-conventions
description: Non-negotiable Laravel + DDD coding conventions for apps/backend/. Applied to every class written in this project.
---

## Architecture — DDD layer rules

The backend follows Domain-Driven Design. Each bounded context has four layers. The dependency rule is strict:

```
Presentation → Application → Domain ← Infrastructure
```

- `Presentation` (Controllers, Requests, Resources) imports from `Application` only
- `Application` (Commands, Handlers, Queries) imports from `Domain` only
- `Infrastructure` (Repositories) implements `Domain` interfaces
- `Domain` (Models, ValueObjects, Events) imports nothing from other layers

**Violation of this dependency rule is forbidden.**

---

## Controllers — 5 lines maximum

Controllers only: receive request → build command → call handler → return resource.

```php
// correct
public function store(StoreInvoiceRequest $request, CreateInvoiceHandler $handler): InvoiceResource
{
    $invoice = $handler->handle(new CreateInvoiceCommand(
        userId: $request->user()->id,
        amount: new Money($request->validated('amount')),
        dueAt:  Carbon::parse($request->validated('due_at')),
    ));

    return new InvoiceResource($invoice);
}

// forbidden — business logic in controller
public function store(Request $request): JsonResponse
{
    $invoice = Invoice::create([...]);
    $invoice->status = 'pending';
    $invoice->save();
    Mail::send(...);
    return response()->json($invoice);
}
```

---

## Validation — always Form Requests

Every `store` and `update` endpoint has a dedicated Form Request. Never validate in controllers.

- `authorize()` delegates to a Policy
- `rules()` returns all validation rules
- Input is accessed only via `$request->validated()` — never `$request->input()` in controllers

---

## Authorization — always Policies

Never write inline role checks (`if ($user->role === 'admin')`). Every resource has a Policy.

```bash
php artisan make:policy InvoicePolicy --model=Invoice
```

Register in `AppServiceProvider` if not auto-discovered.

---

## Models — explicit fillable, no guarded

```php
// correct
protected $fillable = ['user_id', 'amount', 'status', 'due_at'];

// forbidden
protected $guarded = [];
```

---

## Repository pattern

- Define the interface in `Domain/{Context}/Repositories/`
- Implement with Eloquent in `Infrastructure/{Context}/Repositories/`
- Bind in `AppServiceProvider`: `$this->app->bind(FooRepositoryInterface::class, EloquentFooRepository::class)`
- Handlers receive the interface via constructor injection — never instantiate repositories directly

---

## Value Objects

- `readonly` classes
- Validate in the constructor, throw a domain exception if invalid
- Expose value via a named method (`->value()`, `->toString()`, etc.), not a public property

```php
final readonly class Money
{
    public function __construct(private int $amountInCents)
    {
        if ($amountInCents < 0) {
            throw new InvalidMoneyException('Amount cannot be negative.');
        }
    }

    public function value(): int { return $this->amountInCents; }
}
```

---

## Domain Events

- Plain PHP objects (no `ShouldBroadcast` unless needed)
- Named in past tense: `InvoicePaid`, `UserRegistered`
- Dispatched at the **end** of the handler, after persistence
- Listeners registered via `#[AsEventListener]` or in `EventServiceProvider`

---

## API responses — use Resources

Never return `$model->toArray()` or `response()->json($model)`. Always use an API Resource.

---

## Routes

- API: `routes/api.php`, prefix `/api/`, `auth:sanctum` middleware, named routes
- Admin: `routes/web.php`, prefix `/admin`, middleware `['auth', 'role:admin']`
- Route model binding everywhere
- Name every route: `->name('invoices.store')`

---

## Eloquent — avoid N+1

```php
// correct
Invoice::with(['user', 'items'])->paginate();

// forbidden in loops
$invoices->each(fn($i) => $i->user->name);
```

---

## Migrations

- One migration = one logical change
- Never modify a committed migration — create a new one
- Always index foreign keys and frequently filtered columns
- `->nullable()` must be explicit
