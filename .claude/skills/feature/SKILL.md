---
name: feature
description: Scaffold a complete DDD feature end-to-end across the Rendevo stack — from migration to domain, application, infrastructure, presentation layers, tests, and documentation.
argument-hint: "[BoundedContext] [ResourceName] (e.g. Billing Invoice)"
---

The user wants to scaffold a complete feature. Their request: $ARGUMENTS

Follow these steps in order. Show the code for each step before moving to the next.

---

## Step 1 — Clarify scope

Before writing any code, ask if not already provided in $ARGUMENTS:
1. What is the bounded context? (existing or new — e.g. `Billing`, `Catalog`, `Auth`)
2. What is the resource name (singular, English)? e.g. `Invoice`
3. Which layers are needed? (API only / API + admin panel / API + Nuxt frontend / all)
4. What fields does the entity have?
5. Ownership/authorization rules? (e.g. users can only access their own records)
6. Any non-trivial business logic beyond CRUD?

---

## Step 2 — Database

Use the `/migration` skill to create the migration. Pass the table name and fields as arguments.

---

## Step 3 — Domain layer (`app/{Context}/Domain/`)

### 3a. Entity (Eloquent model as domain entity)

File: `app/{Context}/Domain/Models/{Resource}.php`

Create manually or with `php artisan make:model {Resource}` then move to the correct path and update the namespace.

- Explicit `$fillable`, `$casts`, relationships
- Domain-meaningful methods (not just getters/setters)

### 3b. Value Objects (if needed)

File: `app/{Context}/Domain/ValueObjects/{Name}.php`

- `readonly` class, validation in constructor, domain exception on failure

### 3c. Domain Event

File: `app/{Context}/Domain/Events/{Resource}Created.php`

```php
final readonly class {Resource}Created
{
    public function __construct(public {Resource} ${resource}) {}
}
```

### 3d. Repository Interface

File: `app/{Context}/Domain/Repositories/{Resource}RepositoryInterface.php`

```php
interface {Resource}RepositoryInterface
{
    public function findByUser(int $userId): Collection;
    public function findOrFail(int $id): {Resource};
    public function save({Resource} ${resource}): void;
}
```

### 3e. Factory

File: `database/factories/{Resource}Factory.php`

> If organizing by context (`database/factories/{Context}/`), declare `protected static function newFactory()` in the model and register the namespace in `composer.json` autoload-dev.

---

## Step 4 — Application layer (`app/{Context}/Application/`)

### 4a. Command DTO

```php
// app/{Context}/Application/Commands/Create{Resource}Command.php
final readonly class Create{Resource}Command
{
    public function __construct(
        public int $userId,
        // ... typed fields
    ) {}
}
```

### 4b. Command Handler (use case)

```php
// app/{Context}/Application/Handlers/Create{Resource}Handler.php
final class Create{Resource}Handler
{
    public function __construct(
        private {Resource}RepositoryInterface $repository,
    ) {}

    public function handle(Create{Resource}Command $command): {Resource}
    {
        // 1. Instantiate domain objects
        // 2. Persist via repository
        // 3. Dispatch domain event
        event(new {Resource}Created($entity));
        return $entity;
    }
}
```

### 4c. Query (read side, if needed)

Queries can call Eloquent directly — no repository interface needed for reads.

---

## Step 5 — Infrastructure layer (`app/{Context}/Infrastructure/`)

File: `app/{Context}/Infrastructure/Repositories/Eloquent{Resource}Repository.php`

Implements `{Resource}RepositoryInterface`. Bind in `AppServiceProvider`:

```php
$this->app->bind({Resource}RepositoryInterface::class, Eloquent{Resource}Repository::class);
```

---

## Step 6 — Presentation layer (API)

Files in `app/Presentation/Api/`:

```bash
php artisan make:request Presentation/Api/Requests/Store{Resource}Request
php artisan make:request Presentation/Api/Requests/Update{Resource}Request
php artisan make:policy {Resource}Policy --model={Resource}
php artisan make:resource Presentation/Api/Resources/{Resource}Resource
```

Controller at `app/Presentation/Api/Controllers/{Resource}Controller.php` — max 5 lines per method, delegates entirely to handlers.

Register in `routes/api.php`.

---

## Step 7 — Admin panel (`inertia-admin` agent, if requested)

- Controller at `app/Presentation/Admin/Controllers/{Resource}Controller.php`
- Reuse the same Application handlers — no duplicated logic
- Vue pages: `resources/js/Pages/Admin/{Resource}/Index.vue`, `Form.vue`
- Register in `routes/web.php` under `/admin`

---

## Step 8 — Nuxt frontend (`nuxt-frontend` agent, if requested)

- `app/pages/{resource}/index.vue` — list with loading/empty/error states
- `app/pages/{resource}/[id].vue` — detail view
- `app/composables/use{Resource}s.ts` — API calls
- Types in `packages/shared-types/`

---

## Step 9 — Tests

Use the `/test` skill targeting the controller and handler just created. It will generate both the Feature test and the Handler unit test.

---

## Step 10 — Feature documentation

The product manager already created `docs/features/{context}/{feature}.md` with the business sections. Fill in the technical sections: endpoints, handlers, domain events, known limitations.

If the product manager has not yet created the file, create it from `docs/features/_template.md` and fill in all sections.

---

## Step 11 — Summary

Output a checklist of all files created/modified.
