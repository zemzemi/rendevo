---
name: migration
description: Create a complete Laravel database migration following project conventions — correct naming, column types, indexes, rollback, and model update checklist.
argument-hint: "[description of what the migration should do]"
---

The user wants to create a database migration. Their request: $ARGUMENTS

## Step 1 — Clarify if needed

If $ARGUMENTS is ambiguous, ask:
1. What table is being created or modified?
2. What columns are being added, changed, or removed?
3. Are there foreign key relationships?
4. Is this a new table or altering an existing one?
5. Any specific indexes needed beyond foreign keys?

---

## Step 2 — Name the migration correctly

```bash
# New table
php artisan make:migration create_{table}_table

# Add columns
php artisan make:migration add_{columns}_to_{table}_table

# Modify column
php artisan make:migration change_{column}_in_{table}_table

# Drop column
php artisan make:migration remove_{column}_from_{table}_table

# Pivot table
php artisan make:migration create_{table_a}_{table_b}_table
```

---

## Step 3 — Write the migration

### Column conventions

- Always `id()` and `timestamps()` on new tables
- Foreign keys: `foreignId('user_id')->constrained()->cascadeOnDelete()`
- Nullable must be explicit: `->nullable()`
- Enums: use `->string('status')` + a domain enum class, not `->enum()` (avoids DB-level enum lock-in)
- Money: store as integer (cents) — `->unsignedInteger('amount_in_cents')`
- Soft deletes: `->softDeletes()` if the entity needs recovery
- UUID primary key if needed: `$table->uuid('id')->primary()`

### Index conventions

Always add indexes for:
- Every foreign key column (automatically added by `foreignId()->constrained()`)
- Columns used in `WHERE` clauses (status, type, date ranges)
- Columns used in `ORDER BY` on large tables
- Unique constraints where applicable

```php
$table->index(['user_id', 'status']); // composite for common query pattern
$table->index('due_at');
$table->unique(['user_id', 'plan_id']); // if relevant
```

### Rollback

Every migration must have a complete `down()` method. Never leave it empty.

### Full example — new table

```php
public function up(): void
{
    Schema::create('invoices', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->cascadeOnDelete();
        $table->string('number')->unique();
        $table->string('status')->default('draft');
        $table->unsignedInteger('amount_in_cents');
        $table->string('currency', 3)->default('EUR');
        $table->text('notes')->nullable();
        $table->date('due_at')->nullable();
        $table->timestamp('paid_at')->nullable();
        $table->timestamps();
        $table->softDeletes();

        $table->index(['user_id', 'status']);
        $table->index('due_at');
    });
}

public function down(): void
{
    Schema::dropIfExists('invoices');
}
```

---

## Step 4 — Flag model updates needed

After writing the migration, list any changes required in the domain model:
- New columns to add to `$fillable`
- New `$casts` entries (dates, booleans, enums)
- New relationships to define

---

## Step 5 — Output

1. The `php artisan make:migration` command to run
2. The complete migration file content
3. The model changes needed (`$fillable`, `$casts`, relationships)
