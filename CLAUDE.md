# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rendevo is a monorepo with a Laravel 12 backend and a Nuxt 4 frontend. Each app has its own CLAUDE.md with detailed guidance:

- **Backend**: [`apps/backend/CLAUDE.md`](apps/backend/CLAUDE.md)
- **Frontend**: [`apps/frontend/CLAUDE.md`](apps/frontend/CLAUDE.md)

```
rendevo/
├── apps/
│   ├── backend/        # Laravel 12 (PHP 8.4) — REST API + Inertia admin
│   └── frontend/       # Nuxt 4 / Vue 3 / TypeScript — customer SPA
├── packages/
│   └── shared-types/   # Shared TypeScript types
├── docker-compose.yml  # Dev environment (backend, frontend, queue, MySQL)
└── package.json        # npm workspaces root
```

## Quick Start

### With Docker (recommended)

```bash
docker-compose up -d        # Start all services
docker-compose exec backend php artisan migrate  # First-time: run migrations
```

| Service | URL |
|---|---|
| Backend API | http://localhost:8000 |
| API Docs | http://localhost:8000/docs/api |
| Frontend | http://localhost:3000 |
| MySQL | localhost:3306 |

```bash
docker-compose down          # Stop all services
docker-compose down -v       # Stop + reset database
docker-compose logs -f       # Follow logs
```

### Without Docker (local)

```bash
# Backend (from apps/backend/)
composer setup   # First-time setup
composer dev     # Start all services (server + queue + logs + vite)
```

## How the Apps Connect

The backend and frontend are independent services that communicate over HTTP. In Docker, all services share the `rendevo` network. Outside Docker, `composer dev` starts both concurrently.

**Important for Docker:** do not have `node_modules/` inside `apps/frontend/` on the host — Node dependencies live exclusively in the container.
