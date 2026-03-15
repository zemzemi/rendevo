# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### In Docker (recommended)

```bash
docker-compose up -d frontend               # Start frontend service
docker-compose logs -f frontend             # Follow logs (http://localhost:3000)
docker-compose exec frontend npm run lint   # ESLint
docker-compose exec frontend npm run typecheck  # TypeScript check
```

> **Note:** do not run `npm install` inside `apps/frontend/` on the host — Node dependencies must stay in the Docker container. Running npm on the host will create `node_modules/` with macOS binaries that break the container.

### Local (without Docker)

```bash
npm run dev          # Start Nuxt dev server (with DevTools enabled)
npm run build        # Production build
npm run postinstall  # nuxt prepare — regenerates .nuxt/ types after adding modules
npm run lint         # ESLint — check Vue/TypeScript
npm run lint:fix     # ESLint — auto-fix
npm run typecheck    # TypeScript strict type check
```

## Architecture

**Stack**: Nuxt 4.4.2, Vue 3.5.30, Vue Router 5, TypeScript.

**Entry point**: `app/app.vue` — root component wrapping `<NuxtRouteAnnouncer />` and `<NuxtPage />`.

**File-based routing**: Pages go in `app/pages/` and are auto-routed by Nuxt. Currently no pages exist — the app renders Nuxt's default welcome screen.

**TypeScript**: Config in `tsconfig.json` references auto-generated type files under `.nuxt/` (do not edit those files manually).

**Nuxt config** (`nuxt.config.ts`): Minimal setup — `compatibilityDate: '2025-07-15'`, DevTools enabled. Add modules, runtimeConfig, and other settings here.

## Nuxt 4 Conventions

- App directory structure: `app/pages/`, `app/components/`, `app/composables/`, `app/layouts/`, `app/middleware/`, `app/plugins/`
- Server routes (API): `server/api/` and `server/routes/`
- Auto-imports: components, composables, and utils are auto-imported — no need for explicit imports.
- Run `npm run postinstall` after adding Nuxt modules to regenerate `.nuxt/` types.
