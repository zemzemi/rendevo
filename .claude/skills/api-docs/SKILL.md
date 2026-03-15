---
name: api-docs
description: Generate or audit the Scramble API documentation. Checks that controllers have correct return types and resources have explicit toArray() returns.
argument-hint: "[generate | audit | path/to/controller]"
disable-model-invocation: true
---

The user wants to work with API documentation. Action: $ARGUMENTS

## Current API routes

!`cd /Users/adel/www/apps/rendevo/apps/backend && php artisan route:list --path=api --columns=method,uri,name,action 2>/dev/null | head -50 || echo "Run from apps/backend/"`

---

## Mode: generate

If $ARGUMENTS is `generate` or empty, export the OpenAPI spec:

```bash
cd apps/backend
php artisan scramble:export
```

Then verify:
- Docs UI: `http://localhost:8000/docs/api`
- OpenAPI spec: `http://localhost:8000/docs/api.json`

---

## Mode: audit

If $ARGUMENTS is `audit`, review all API controllers in `app/Presentation/Api/Controllers/` and check each endpoint:

**Controller methods:**
- [ ] Return type explicitly declared? (`InvoiceResource`, `AnonymousResourceCollection`, `JsonResponse`, `Response`)
- [ ] No `mixed` or missing return type?
- [ ] Route model binding used where applicable?
- [ ] Short PHPDoc description present for non-obvious endpoints?

**API Resources:**
- [ ] `toArray()` returns an explicit array (no dynamic keys, no `array_merge()` with unknown shapes)?
- [ ] All returned fields have inferrable types (no untyped closures or dynamic values)?

**Form Requests:**
- [ ] `rules()` uses typed validation rules Scramble can read? (`required`, `string`, `integer`, `boolean`, `date`, `nullable`, etc.)

Report missing or ambiguous types with the exact file and method.

---

## Mode: endpoint (specific file)

If $ARGUMENTS contains a controller path, audit that controller only:

1. Read the controller, its Form Request(s), and its API Resource(s)
2. List what Scramble cannot infer (missing return types, dynamic arrays)
3. Provide corrected files with explicit types added

---

## Common fixes

### Missing return type on controller method
```php
// before — Scramble cannot document this
public function store(StoreInvoiceRequest $request, CreateInvoiceHandler $handler)

// after — explicit return type
public function store(StoreInvoiceRequest $request, CreateInvoiceHandler $handler): InvoiceResource
```

### Dynamic toArray() Scramble cannot read
```php
// before — Scramble cannot infer the shape
public function toArray(Request $request): array
{
    return array_merge(parent::toArray($request), ['extra' => $this->extra]);
}

// after — explicit array
public function toArray(Request $request): array
{
    return [
        'id'     => $this->id,
        'status' => $this->status,
        'extra'  => $this->extra,
    ];
}
```

---

## Output

Always end with:
- Count of endpoints fully documented vs total
- List of methods with missing or ambiguous types
- Command to regenerate: `php artisan scramble:export`
