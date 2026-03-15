---
name: api-docs
description: Conventions for documenting the Laravel API with Scramble. No annotations needed — Scramble reads Form Requests, API Resources, and return types via static analysis.
---

API documentation is generated automatically by **Dedoc/Scramble** (`php artisan scramble:export`).

- UI: `http://localhost:8000/docs/api`
- OpenAPI spec: `http://localhost:8000/docs/api.json`

Scramble requires **no PHPDoc annotations**. It reads the code directly:

| Code | What Scramble infers |
|---|---|
| Form Request `rules()` | Request body parameters and types |
| API Resource `toArray()` | Response shape and field types |
| Controller return type hints | Response type |
| Route model binding | URL parameters |
| `->middleware('auth:sanctum')` | Authenticated endpoint |

---

## What Scramble reads automatically

### Form Request → request body

```php
public function rules(): array
{
    return [
        'amount_in_cents' => ['required', 'integer', 'min:1'],
        'currency'        => ['string', 'size:3'],
        'due_at'          => ['required', 'date'],
        'notes'           => ['nullable', 'string', 'max:500'],
    ];
}
```

Scramble generates the full request body schema from this — no extra work needed.

### API Resource → response shape

```php
public function toArray(Request $request): array
{
    return [
        'id'              => $this->id,
        'status'          => $this->status,
        'amount_in_cents' => $this->amount_in_cents,
        'currency'        => $this->currency,
        'due_at'          => $this->due_at?->toDateString(),
        'created_at'      => $this->created_at->toISOString(),
    ];
}
```

Scramble infers types from the return array — keep return types explicit.

### Controller → endpoint metadata

```php
// Return type required — Scramble uses it to document the response
public function store(StoreInvoiceRequest $request, CreateInvoiceHandler $handler): InvoiceResource
{
    $entity = $handler->handle(new CreateInvoiceCommand(...));
    return new InvoiceResource($entity);
}

// For collections — use the typed collection return
public function index(Request $request): AnonymousResourceCollection
{
    return InvoiceResource::collection(
        Invoice::where('user_id', $request->user()->id)->paginate()
    );
}
```

---

## PHPDoc — only when types are ambiguous

Scramble reads PHPDoc only when it cannot infer the type statically. Use it sparingly:

```php
/**
 * Cancel an invoice.
 *
 * Marks the invoice as cancelled and triggers a refund if already paid.
 */
public function cancel(Invoice $invoice, CancelInvoiceHandler $handler): InvoiceResource
```

A description is always welcome. Type annotations are only needed for dynamic return types that Scramble cannot resolve.

---

## Rules

- **Return types are mandatory** on every controller method — Scramble cannot document what it cannot infer
- **Keep `toArray()` return explicit** in API Resources — avoid `array_merge()` or dynamic keys that Scramble cannot read
- **No PHPDoc annotations for types** — let static analysis do its job; add PHPDoc only for descriptions
- **Run `php artisan scramble:export`** after every new or modified endpoint
- Docs UI is available in development only — restrict in production via `scramble.php` middleware config
