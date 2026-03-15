---
name: shadcn-vue
description: Conventions for using shadcn-vue in both the Nuxt frontend (apps/frontend/) and the Inertia admin panel (apps/backend/resources/js/).
---

shadcn-vue is the UI component library for this project. Components are copied into the codebase — they are not a npm dependency.

## Installation

```bash
# Nuxt frontend (apps/frontend/)
npx shadcn-vue@latest init
npx shadcn-vue@latest add <component>

# Inertia admin (apps/backend/)
npx shadcn-vue@latest init
npx shadcn-vue@latest add <component>
```

## Component locations

```
apps/frontend/app/
└── components/
    └── ui/          # shadcn-vue generated components — DO NOT edit manually
    └── ...          # custom app components

apps/backend/resources/js/
└── Components/
    └── ui/          # shadcn-vue generated components — DO NOT edit manually
    └── ...          # custom admin components
```

## Rule: never manually edit `components/ui/`

Files in `components/ui/` are owned by the shadcn-vue CLI. Manual edits will be overwritten on the next `add` or update. To customize a component, **wrap it** in a new component outside `ui/`.

```vue
<!-- correct: wrap in a custom component -->
<!-- components/AppButton.vue -->
<script setup lang="ts">
import { Button } from '@/components/ui/button'
</script>
<template>
  <Button v-bind="$attrs"><slot /></Button>
</template>

<!-- forbidden: editing components/ui/button/Button.vue directly -->
```

## Rule: always use `cn()` for conditional classes

`cn()` combines `clsx` and `tailwind-merge`. It prevents class conflicts and is the standard across all shadcn components.

```ts
import { cn } from '@/lib/utils'

// correct
const classes = cn('px-4 py-2', isActive && 'bg-primary', props.class)

// forbidden — class conflicts will silently lose
const classes = `px-4 py-2 ${isActive ? 'bg-primary' : ''} ${props.class}`
```

## Rule: use CVA for custom component variants

When building custom components that need variants (size, intent, state), use `class-variance-authority` — the same pattern shadcn uses internally.

```ts
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const badgeVariants = cva('inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-semibold', {
  variants: {
    intent: {
      default:     'bg-primary text-primary-foreground',
      success:     'bg-green-100 text-green-800',
      warning:     'bg-yellow-100 text-yellow-800',
      destructive: 'bg-destructive text-destructive-foreground',
    },
  },
  defaultVariants: { intent: 'default' },
})

export type BadgeVariants = VariantProps<typeof badgeVariants>
```

## Rule: use CSS variables for colors — never hardcoded Tailwind palette

shadcn-vue themes are built on CSS variables. Always use semantic tokens so dark mode and theming work correctly.

```html
<!-- correct — respects theme -->
<div class="bg-background text-foreground border-border">
<Button class="bg-primary text-primary-foreground">

<!-- forbidden — breaks dark mode and theming -->
<div class="bg-white text-gray-900 border-gray-200">
<Button class="bg-blue-600 text-white">
```

Available semantic tokens: `background`, `foreground`, `primary`, `primary-foreground`, `secondary`, `secondary-foreground`, `muted`, `muted-foreground`, `accent`, `accent-foreground`, `destructive`, `destructive-foreground`, `border`, `input`, `ring`, `card`, `card-foreground`, `popover`, `popover-foreground`.

## Rule: import from the component index, not the file directly

```ts
// correct — one import per component module
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'

// forbidden — importing directly from the .vue file
import Button from '@/components/ui/button/Button.vue'
```

## Theming

Customize the theme in the global CSS file (not in component files):

- Nuxt: `app/assets/css/main.css`
- Admin: `resources/css/app.css`

Override CSS variables inside `:root` and `.dark` — never touch `components/ui/`.
