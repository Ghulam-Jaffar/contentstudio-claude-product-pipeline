# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ContentStudio** — a full-stack social media management platform for composing, scheduling, publishing, and analyzing content across Facebook, Instagram, LinkedIn, Twitter/X, TikTok, Pinterest, YouTube, Medium, Tumblr, and WordPress.

Two independent sub-projects live in this repo:
- `contentstudio-backend/` — Laravel 10 API (PHP 8.3, MongoDB, Redis, Kafka)
- `contentstudio-frontend/` — Vue 3 SPA (Composition API, Vuex → Pinia migration)

Each has its own `CLAUDE.md` with project-specific rules. **Read the sub-project CLAUDE.md before working in that codebase.**

## Development Commands

### Backend (`contentstudio-backend/`)

All commands run through Laravel Sail (Docker). Prefix everything with `vendor/bin/sail`.

```bash
# Environment
vendor/bin/sail up -d                    # Start Docker services
vendor/bin/sail stop                     # Stop services

# Development
vendor/bin/sail artisan migrate          # Run migrations
vendor/bin/sail artisan horizon          # Start queue worker
vendor/bin/sail npm run dev              # Frontend hot reload (backend assets)

# Testing (Pest v2)
vendor/bin/sail artisan test --compact                              # All tests
vendor/bin/sail artisan test --compact tests/Feature/SomeTest.php   # Single file
vendor/bin/sail artisan test --compact --filter=testName            # Single test

# Code quality
vendor/bin/sail bin pint --dirty         # Format changed PHP files (Pint)
./vendor/bin/phan --allow-polyfill-parser   # Static analysis (Phan)
./vendor/bin/phpstan analyse --memory-limit=2G  # Static analysis (Larastan)

# Scaffolding (always pass --no-interaction)
vendor/bin/sail artisan make:test --pest {name}
vendor/bin/sail artisan make:model {name}
vendor/bin/sail artisan make:controller {name}
```

### Frontend (`contentstudio-frontend/`)

```bash
# Environment
yarn install
doppler setup                # Select project: contentstudio-frontend, config: dev_latest

# Development
yarn dev                     # Dev server with hot reload
yarn build                   # Production build

# Testing
yarn test:unit               # Jest unit tests
yarn test:e2e                # Cypress e2e tests
yarn test:unit:watch         # Watch mode

# Linting (run all: yarn lint)
yarn biome:check             # Biome — primary linter/formatter
yarn lint:eslint             # ESLint
yarn lint:stylelint          # Stylelint (SCSS/CSS)
yarn prettier:format         # Prettier

# Code generation (Hygen)
yarn new component           # New component + test
yarn new view                # New view
yarn new module              # New Vuex module
```

## Architecture

### Backend

**Laravel 10 with service-layer architecture:**
- `app/Http/Controllers/` — RESTful API controllers grouped by domain (Accounts, AI, Analytics, Composer, etc.)
- `app/Services/` + `app/Helpers/` — business logic layer
- `app/Repositories/` + `app/Repository/` — data access abstraction
- `app/Builders/` — 29 builder classes for complex object construction
- `app/Jobs/` — queued background jobs (Redis + Kafka via `mateusjunges/laravel-kafka`)
- `app/Events/` + `app/Listeners/` — event-driven architecture
- `app/Strategy/` — strategy pattern implementations
- `app/Models/` — Eloquent ORM over MongoDB (`mongodb/laravel-mongodb`)

**Routing:** `routes/api/v1.php` (main API), `routes/web/` (per-domain route files), `routes/billing.php`, `routes/mcp.php`

**Key Laravel 10 note:** Use `protected $casts = []` property (not the `casts()` method). Middleware in `app/Http/Kernel.php`, exceptions in `app/Exceptions/Handler.php`, schedules in `app/Console/Kernel.php`.

**Database:** MongoDB (primary), Redis (caching/queues), Clickhouse (analytics)

### Frontend

**Vue 3 SPA with 19 feature modules:**
- `src/modules/` — self-contained feature modules (account, analytics_v3, AI-tools, automation, billing, composer_v2, discovery, inbox-revamp, integration, onboarding, planner_v2, publish, publisher, setting, etc.)
- Each module has its own components, composables, routes (`config/routes/`), and store
- `src/composables/` — shared Vue 3 composition functions
- `src/store/` — Vuex namespaced modules (migrating to Pinia/composables)
- `src/components/` — shared components (common, globals, layout, UI)
- `src/config/api-utils.js` — centralized API URL management

**Path aliases:** `@` (root), `@src`, `@assets`, `@state`, `@common`, `@ui`, `@modules`

**HTTP:** Always use proxy from `@common/lib/http-common` — never raw axios/fetch.

**UI:** `@contentstudio/ui` component library + Tailwind CSS. Global modal: `$cstuModal.show/hide('id')`.

**EventBus:** `@common/lib/event-bus` — temporary pattern, always clean up with `$off` in `onUnmounted`.

## Cross-Project Conventions

- **Search the repo first** before implementing any pattern — match existing conventions
- **Backend:** Follow Spatie/Laravel conventions (PSR-12, early returns, constructor promotion, typed properties). Format with Pint before finalizing changes
- **Frontend:** `<script setup>` mandatory for new components. Tailwind first, minimal SCSS. Composables over Vuex for new code
- **Testing is required** for all changes. Backend: Pest (feature tests preferred). Frontend: Jest + Cypress
- **No hardcoded URLs** — backend uses `config()` + named routes; frontend uses `api-utils.js`
- **No `env()` outside config files** (backend). No raw axios/fetch (frontend)
- **Stick to existing directory structure** — don't create new base folders without approval
- **Don't change dependencies** without approval

## Frontend Modernization Phases (2026)

The frontend follows a strict phased approach — never jump ahead:
1. **Dead code & redundancy** — remove unused files, redundant packages (jQuery leftovers, lodash.*, etc.)
2. **Memory & perf** — cleanup in `onUnmounted` (timers, listeners, Pusher), replace EventBus misuse, refactor large components
3. **Dependencies** — upgrade socket.io, pusher-js, @sentry/vue; remove jQuery/Bootstrap → Tailwind; Moment.js → Day.js
4. **Framework upgrades** — Vue, Router, VueUse, ESLint version bumps
