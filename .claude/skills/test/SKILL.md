---
name: test
description: Generate missing tests for existing code — Feature tests (HTTP level) and Handler unit tests (fake repository). Covers controllers, handlers, domain models, and value objects.
argument-hint: "[path/to/file or ClassName]"
---

The user wants to generate tests for existing code. Target: $ARGUMENTS

## Step 1 — Read the target code

Read the file(s) specified in $ARGUMENTS in full. Identify:
- What layer it belongs to (Controller, Handler, Domain model, Value Object, Repository)
- What it does
- What inputs it accepts and what outputs/side effects it produces
- What can go wrong (validation, authorization, missing records, invariant violations)

If $ARGUMENTS is a controller or route, also read the associated Handler, Form Request, and Policy.

---

## Step 2 — Determine which test types are needed

| Code being tested | Test type | Location |
|---|---|---|
| API Controller (route) | Feature test | `tests/Feature/Api/{Resource}Test.php` |
| Admin Controller (Inertia) | Feature test | `tests/Feature/Admin/{Resource}Test.php` |
| Application Handler | Unit test | `tests/Unit/{Context}/Application/{Handler}Test.php` |
| Domain Model method | Unit test | `tests/Unit/{Context}/Domain/{Model}Test.php` |
| Value Object | Unit test | `tests/Unit/{Context}/Domain/{ValueObject}Test.php` |

Generate both types when both are missing.

---

## Step 3 — Feature test (HTTP level)

Uses `RefreshDatabase` + real in-memory SQLite. Tests the full request → response → DB cycle.

Required cases for every endpoint:

```php
public function test_{action}_{resource}_successfully(): void         // happy path
public function test_unauthenticated_user_cannot_{action}(): void     // → 401
public function test_user_cannot_{action}_{resource}_they_do_not_own(): void // → 403
public function test_{action}_fails_with_invalid_data(): void         // → 422
public function test_{action}_returns_404_when_not_found(): void      // show/update/destroy only
```

---

## Step 4 — Handler unit test (application layer)

Uses a fake in-memory repository — no DB, no HTTP. The fake class lives in `tests/Fakes/{Context}/InMemory{Resource}Repository.php` (create it if missing, see `testing-policy` rule for the template).

```php
class {Handler}Test extends TestCase
{
    private {RepositoryInterface} $repository;
    private {Handler} $handler;

    protected function setUp(): void
    {
        parent::setUp();
        $this->repository = new InMemory{Resource}Repository(); // tests/Fakes/{Context}/
        $this->handler = new {Handler}($this->repository);
    }

    public function test_handler_creates_{resource}(): void
    {
        $result = $this->handler->handle(new {Command}(/* valid data */));

        $this->assertInstanceOf({Resource}::class, $result);
        $this->assertCount(1, $this->repository->all());
    }

    public function test_handler_dispatches_{event}_event(): void
    {
        Event::fake();
        $this->handler->handle(new {Command}(/* valid data */));
        Event::assertDispatched({Event}::class);
    }

    public function test_handler_throws_when_{invariant_violated}(): void
    {
        $this->expectException({DomainException}::class);
        $this->handler->handle(new {Command}(/* invalid data */));
    }
}
```

---

## Step 5 — Value Object / Domain model unit test

```php
class {ValueObject}Test extends TestCase
{
    public function test_accepts_valid_value(): void { ... }
    public function test_rejects_invalid_value(): void { ... }  // adapt to the invariant
    public function test_equality(): void { ... }               // if applicable
}
```

---

## Step 6 — Output

1. List of all test files to create with their paths
2. Complete content of each test file
3. Any factory or fake repository missing that the tests need
