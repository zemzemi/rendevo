---
name: ux
description: UX rules that apply to every screen in both the Nuxt frontend and the Inertia admin panel. These are non-negotiable interface standards, not suggestions.
---

## Every screen must have a clear primary action

Each view has one — and only one — visually dominant action. Use the `default` or `primary` Button variant for it. All secondary actions use `secondary`, `outline`, or `ghost`.

```vue
<!-- correct -->
<Button>Create invoice</Button>
<Button variant="outline">Cancel</Button>

<!-- forbidden — two competing primary actions -->
<Button>Save</Button>
<Button>Export</Button>
```

---

## Every async screen must define loading, empty, and error states

No screen that fetches data can ship without all three states handled.

**Nuxt (Composition API):**

```vue
<script setup lang="ts">
const { data, status, error } = await useFetch('/api/invoices')
</script>

<template>
  <div v-if="status === 'pending'">
    <Skeleton class="h-10 w-full" />
  </div>
  <div v-else-if="error">
    <Alert variant="destructive">
      <AlertDescription>Failed to load invoices. Please try again.</AlertDescription>
    </Alert>
  </div>
  <div v-else-if="!data?.length">
    <EmptyState title="No invoices yet" description="Create your first invoice to get started." />
  </div>
  <div v-else>
    <!-- actual content -->
  </div>
</template>
```

**Inertia (props-driven):** handle empty and error states via props passed from the controller. Use Laravel's `optional()` and flash messages for errors.

---

## Forms must show validation messages clearly

- Display field-level errors directly below the relevant input, not only at the top of the form
- Use the `FormMessage` component from shadcn-vue
- Never rely on browser-native validation alone (`required`, `type="email"`)
- Show errors after the first submit attempt, not on every keystroke

```vue
<FormField name="email">
  <FormItem>
    <FormLabel>Email</FormLabel>
    <FormControl>
      <Input type="email" v-bind="field" />
    </FormControl>
    <FormMessage /> <!-- renders the error for this field -->
  </FormItem>
</FormField>
```

For Inertia forms, use `form.errors.fieldName` from `useForm()`:

```vue
<Input v-model="form.email" />
<p v-if="form.errors.email" class="text-sm text-destructive">{{ form.errors.email }}</p>
```

---

## Destructive actions must require confirmation

Any action that deletes data, cancels a subscription, or is otherwise irreversible must show a confirmation dialog before executing.

Use `AlertDialog` from shadcn-vue — not `window.confirm()`.

```vue
<AlertDialog>
  <AlertDialogTrigger as-child>
    <Button variant="destructive">Delete invoice</Button>
  </AlertDialogTrigger>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Are you sure?</AlertDialogTitle>
      <AlertDialogDescription>This invoice will be permanently deleted.</AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Cancel</AlertDialogCancel>
      <AlertDialogAction @click="deleteInvoice" class="bg-destructive text-destructive-foreground">
        Delete
      </AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

---

## Tables and dashboards must stay readable on laptop screens

- Default to a readable column count — do not show all available fields
- Hide secondary columns below `lg` breakpoint using `class="hidden lg:table-cell"`
- Long text in cells: truncate with `truncate max-w-[200px]` and show full value in a tooltip
- Use `DataTable` from shadcn-vue with pagination — never render unbounded lists
- Sticky header on long tables: `<TableHeader class="sticky top-0 bg-background z-10">`

---

## Avoid modal overuse

Modals interrupt the user's flow. Use them only when the action is short, self-contained, and does not require navigation.

| Use a modal for | Use a page instead |
|---|---|
| Quick edit of 1–3 fields | Full create/edit forms |
| Confirmation dialogs | Multi-step workflows |
| Preview / detail peek | Settings or profile pages |
| Filters on a table | Complex filtering with many options |

When in doubt, use a page. Navigation is cheaper than a poorly designed modal.
