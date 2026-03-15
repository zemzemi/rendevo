---
name: nuxt-conventions
description: Non-negotiable Vue 3 / Nuxt 4 / TypeScript conventions for apps/frontend/ and the Inertia admin panel (apps/backend/resources/js/). Applied to every component and composable written in this project.
---

## Shared types

API response types are defined in `packages/shared-types/` — never redefined locally.

```ts
// correct
import type { Invoice } from '@rendevo/shared-types'

// forbidden — redefining a type that already exists in shared-types
interface Invoice { id: number; status: string }
```

New types are added by the `tech-lead` — coordinate before creating anything in `packages/shared-types/src/`.

---

## TypeScript — strict, no `any`

```ts
// correct
const user = ref<User | null>(null)
function getUser(id: unknown): User {
  if (typeof id !== 'number') throw new Error('Invalid id')
  ...
}

// forbidden
const user: any = null
function getUser(id: any) { ... }
```

- Props: always `defineProps<{ label: string; count?: number }>()`
- Emits: always `defineEmits<{ submit: [value: string] }>()`
- No `any` — use `unknown` and narrow with type guards

---

## Composables — SSR-safe

Composables must be safe to run on the server. Never access browser globals at module level.

```ts
// correct — guarded
const isClient = typeof window !== 'undefined'
onMounted(() => { window.scrollTo(0, 0) })

// forbidden — crashes on SSR
const width = window.innerWidth
```

- Prefix with `use`: `useInvoices`, `useAuth`
- Return reactive state (`ref`, `computed`) — not raw values
- Side-effect-free where possible; isolate effects in `onMounted` / watchers

---

## API calls — `useFetch` / `useAsyncData` only

Never use raw `fetch` or `axios` in `<script setup>`. Nuxt's data-fetching composables handle SSR, deduplication, and error states.

```ts
// correct
const { data, status, error } = await useFetch('/api/invoices')

// forbidden
const res = await fetch('/api/invoices')
const data = await res.json()
```

For imperative calls (e.g. form submissions), use `$fetch`.

---

## Nuxt pages — `definePageMeta()`

Every page that needs auth, a specific layout, or middleware must declare it explicitly:

```ts
definePageMeta({
  middleware: 'auth',
  layout: 'dashboard',
})
```

---

## State — `useState` vs Pinia

| Use case | Tool |
|---|---|
| SSR-safe shared state (small) | `useState()` |
| Complex global state with actions | Pinia store |
| Local component state | `ref` / `reactive` |

Never use `localStorage` directly for state shared across components — use Pinia with a persistence plugin if needed.

---

## Inertia conventions (admin panel)

- **Forms**: `useForm()` from `@inertiajs/vue3` — never raw `axios` or `$fetch`
- **Links**: `<Link>` from `@inertiajs/vue3` — never `<a href>` or `<router-link>`
- **Layouts**: `defineOptions({ layout: AdminLayout })` in page components
- **Flash messages**: read from `usePage().props.flash` — set via `session()->flash()` in Laravel

---

## Security in templates

```vue
<!-- correct — escaped output -->
<p>{{ userInput }}</p>
<p v-text="userInput" />

<!-- forbidden — XSS risk with user-generated content -->
<p v-html="userInput" />
```

`v-html` is only acceptable with content that is guaranteed to be sanitized server-side and never comes from user input.

---

## Forbidden patterns

- `v-html` with user-generated content → XSS
- `any` in TypeScript → silent type errors
- Raw `fetch` in `<script setup>` → breaks SSR
- `window`/`document` at module level in composables → crashes SSR
- Hardcoded Tailwind palette colors (`bg-blue-600`) → breaks theming (use CSS tokens, see `rules/shadcn-vue.md`)
