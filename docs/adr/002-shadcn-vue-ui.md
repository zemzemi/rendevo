# ADR-002: shadcn-vue as the UI component library

**Status**: `accepted`
**Date**: 2026-03-14
**Deciders**: Adel

## Context

Both the Nuxt frontend and the Inertia admin panel need a consistent component library. The choice must support Vue 3, Tailwind CSS 4, TypeScript, and be usable in both Nuxt and Inertia contexts without conflict.

## Decision

Use **shadcn-vue** (the Vue port of shadcn/ui) for all UI components in both apps.

Components are copied into the codebase via CLI (`npx shadcn-vue@latest add`) — not installed as a package dependency. They live in `components/ui/` and are never edited manually.

## Consequences

### Positive
- Full ownership of component code — no breaking changes from upstream updates
- Tailwind CSS + CSS variables: theming and dark mode work out of the box
- Radix Vue primitives ensure accessibility (keyboard nav, ARIA) without extra work
- Same component API in both Nuxt and Inertia — consistent DX across the two apps
- CVA + `cn()` pattern enables consistent variant management for custom components

### Negative
- Components must be re-added via CLI to get upstream updates (not automatic)
- `components/ui/` directory grows with each added component
- Developers must learn the shadcn-vue composition model (FormField, FormItem, etc.)

### Risks
- **Risk**: Developers edit `components/ui/` directly and lose changes on update
- **Mitigation**: Rule enforced in `.claude/rules/shadcn-vue.md` — wrap, never edit

## Alternatives considered

| Alternative | Why rejected |
|---|---|
| PrimeVue | Opinionated styling, harder to customize with Tailwind |
| Naive UI | Vue 3 support but no Tailwind integration |
| Headless UI | Lower-level, requires more work to build a consistent component set |
| Custom components | High cost, inconsistent accessibility |
