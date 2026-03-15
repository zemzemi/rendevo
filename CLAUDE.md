# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rendevo is a monorepo with a Laravel 12 backend and a Nuxt 4 frontend. Each app has its own CLAUDE.md with detailed guidance:

- **Backend**: [`apps/backend/CLAUDE.md`](apps/backend/CLAUDE.md)
- **Frontend**: [`apps/frontend/CLAUDE.md`](apps/frontend/CLAUDE.md)

```
rendevo/
├── apps/
│   ├── backend/        # Laravel 12 (PHP 8.2+) — REST API + Blade
│   └── frontend/       # Nuxt 4 / Vue 3 / TypeScript — SPA
└── packages/
    └── shared-types/   # Shared TypeScript types (currently empty)
```

## Quick Start

```bash
# Backend (from apps/backend/)
composer setup   # First-time setup
composer dev     # Start all services (server + queue + logs + frontend assets)

# Frontend (from apps/frontend/)
npm run dev      # Start Nuxt dev server independently
```

## How the Apps Connect

The backend and frontend are independent services. `composer dev` (run from `apps/backend/`) starts both concurrently. The Nuxt frontend communicates with the Laravel backend over HTTP. There is no SSR coupling or shared session between them.
