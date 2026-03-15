---
name: api-endpoint
description: Generate a single complete REST API endpoint (or CRUD set) following DDD — command, handler, form request, policy, resource, controller, routes, and tests.
argument-hint: "[ResourceName] [methods: index,store,show,update,destroy]"
---

The user wants to generate API endpoint(s). Their request: $ARGUMENTS

## Step 1 — Clarify if needed

Ask if $ARGUMENTS is ambiguous:
1. Bounded context and resource name? (e.g. `Billing` / `Invoice`)
2. Does the domain entity already exist?
3. Which HTTP methods? (`index`, `store`, `show`, `update`, `destroy`)
4. Authentication required? (almost always yes)
5. Ownership scoping? (user sees only their own records?)
6. Any business logic beyond persistence?

---

## Step 2 — Check existing domain

Before generating anything, check if these already exist:
- Domain model: `app/{Context}/Domain/Models/{Resource}.php`
- Repository interface: `app/{Context}/Domain/Repositories/{Resource}RepositoryInterface.php`
- Infrastructure repo: `app/{Context}/Infrastructure/Repositories/Eloquent{Resource}Repository.php`

If missing, generate them first (see `/feature` steps 3–5).

---

## Step 3 — Application layer (one handler per write operation)

**Command DTO** — `app/{Context}/Application/Commands/{Action}{Resource}Command.php`

```php
final readonly class Create{Resource}Command
{
    public function __construct(
        public int $userId,
        // typed fields from validated input
    ) {}
}
```

**Handler** — `app/{Context}/Application/Handlers/{Action}{Resource}Handler.php`

```php
final class Create{Resource}Handler
{
    public function __construct(private {Resource}RepositoryInterface $repository) {}

    public function handle(Create{Resource}Command $command): {Resource}
    {
        // domain logic → persist → dispatch event
        event(new {Resource}Created($entity));
        return $entity;
    }
}
```

For `index`/`show` (reads), query Eloquent directly in the controller — no handler required for simple reads.

---

## Step 4 — Presentation layer

### Form Requests — `app/Presentation/Api/Requests/`

- `authorize()` → delegates to Policy
- `rules()` → full validation; `update` uses `sometimes|required` for PATCH semantics
- Only `$request->validated()` flows to the handler

### Policy

- `viewAny` / `create` → authenticated
- `view` / `update` / `delete` → owner only (if user-owned)

### API Resource — `app/Presentation/Api/Resources/`

- Map only fields needed by the consumer
- No timestamps unless requested
- Nest related resources: `new UserResource($this->user)`

### Controller — `app/Presentation/Api/Controllers/`

```php
final class {Resource}Controller extends Controller
{
    public function store(Store{Resource}Request $request, Create{Resource}Handler $handler): {Resource}Resource
    {
        $entity = $handler->handle(new Create{Resource}Command(
            userId: $request->user()->id,
            // ... $request->validated() fields
        ));
        return new {Resource}Resource($entity);
    }

    public function show({Resource} ${resource}): {Resource}Resource
    {
        $this->authorize('view', ${resource});
        return new {Resource}Resource(${resource});
    }

    public function destroy({Resource} ${resource}, Delete{Resource}Handler $handler): Response
    {
        $this->authorize('delete', ${resource});
        $handler->handle(new Delete{Resource}Command(${resource}));
        return response()->noContent();
    }
}
```

### Routes — `routes/api.php`

```php
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('{resources}', Api\{Resource}Controller::class);
});
```

---

## Step 5 — Scramble compatibility

Follow `rules/api-docs.md`. The two non-negotiable points:

- **Every controller method must have an explicit return type** — Scramble cannot document what it cannot infer
- **`toArray()` in API Resources must return an explicit array** — no `array_merge()`, no dynamic keys

---

## Step 6 — Tests

Feature test: `tests/Feature/Api/{Resource}Test.php` — happy path, 401, 403, 422 for each method.

Handler unit test: `tests/Unit/{Context}/Application/{Action}{Resource}HandlerTest.php` — fake repository from `tests/Fakes/{Context}/`.

---

## Step 7 — Output summary

List every file created/modified with its full path.
Remind to run `php artisan scramble:export` to update the API docs.
