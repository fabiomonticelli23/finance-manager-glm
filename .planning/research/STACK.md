# Technology Stack

**Project:** Finance Manager -- Personal Portfolio Management
**Researched:** 2026-04-20
**Confidence:** HIGH (versions verified against npm registry 2026-04-20)

## Recommended Stack

### Core Framework -- Backend

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **NestJS** | 11.1.x (`@nestjs/core`) | Backend framework | User-specified constraint. v11 is current major. Modular architecture with dependency injection, decorators, and first-class TypeScript support. Excellent ecosystem for scheduled tasks, config, and DB integration. |
| **@nestjs/platform-express** | 11.1.x | HTTP adapter | Default NestJS adapter. Mature, well-documented, no reason to switch to Fastify for a single-user app. |
| **@nestjs/config** | 4.0.x | Configuration management | Type-safe env variable loading with validation. Essential for managing API keys (CCXT, market data) and DB credentials. |
| **@nestjs/schedule** | 6.1.x | Task scheduling | Cron-based auto-polling for CCXT exchange data sync. Supports `@Cron()`, `@Interval()`, and `@Timeout()` decorators. Uses the `cron` package under the hood. |
| **@nestjs/testing** | 11.1.x | Test utilities | NestJS built-in testing module for unit and e2e tests with dependency injection mocking. |

### Core Framework -- Frontend

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **React** | 19.2.x | UI library | User-specified constraint. React 19 is the current stable major. Server components exist but not needed here -- this is a client-side SPA with API calls. |
| **React DOM** | 19.2.x | DOM renderer | Paired with React 19. |
| **Vite** | 8.0.x | Build tool / dev server | The standard for React SPAs in 2025+. Blazing fast HMR, native ESM, zero-config TypeScript. Replaces Create React App (deprecated). |
| **@vitejs/plugin-react** | 6.0.x | Vite React plugin | Official Vite plugin for React with Fast Refresh support. |
| **TypeScript** | 6.0.x | Type system | Shared between frontend and backend. v6 is current. Strict mode enabled for both workspaces. |

### Database

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **PostgreSQL** | 17.x | Relational database | User-specified constraint. Best-in-class relational DB. JSON support for flexible CCXT response caching. Strong indexing for time-series snapshots. |
| **TypeORM** | 0.3.x | ORM | Chosen over Prisma (see Alternatives). Native NestJS integration via `@nestjs/typeorm`. Active Record + Data Mapper patterns. Migration support. Works well with the entity-heavy domain (assets, transactions, snapshots). |
| **@nestjs/typeorm** | 11.0.x | NestJS TypeORM module | Official NestJS wrapper for TypeORM with `forRoot()` / `forFeature()` pattern. |
| **pg** | 8.20.x | PostgreSQL driver | Node.js PostgreSQL client. Required by TypeORM for PostgreSQL connections. |

### Crypto Exchange Integration

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **CCXT** | 4.5.x | Crypto exchange API library | User-specified constraint. Standardized API across 100+ exchanges. `fetchBalance()`, `loadMarkets()`, ticker data all unified. v4 is current major with active maintenance. |

### Market Data & FX Rates

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **yahoo-finance2** | 3.14.x | Stock/ETF/ETP price fetching | Best free option for stock/ETF/ETP market prices. `quote()` returns current price + currency. Community-maintained since 2013, TypeScript support. Beta label but widely used and reliable for personal projects. Server-side only (CORS blocked in browser). |
| **exchange-rate-api** (npm) or **ECB daily rates** via HTTP | 1.x or manual | EUR/USD and multi-currency FX conversion | For FX rates: either use yahoo-finance2 to fetch currency pair quotes (e.g., `EURUSD=X`), or call a free exchange rate API. yahoo-finance2 approach is simpler -- one library for both stocks and FX. |

**Confidence:** HIGH for CCXT and NestJS ecosystem. MEDIUM for yahoo-finance2 (community package, no official Yahoo endorsement -- but there is no better free alternative for personal use).

### Frontend State Management

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **Zustand** | 5.0.x | Client-side state management | Minimal boilerplate, no providers, no reducers. Perfect for a single-user dashboard where global state is portfolio data, allocation targets, and UI filters. Far simpler than Redux for this scope. |
| **@tanstack/react-query** | 5.99.x | Server state / data fetching | Handles API call caching, background refetching, stale-while-revalidate. Essential for keeping portfolio data fresh without manual refresh logic. Replaces manual fetch + useEffect patterns. |

### Frontend UI & Styling

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **Tailwind CSS** | 4.2.x | Utility-first CSS | v4 with Vite plugin is the current standard. `@tailwindcss/vite` for zero-config integration. Utility-first approach is fast for dashboard layouts. |
| **@tailwindcss/vite** | 4.2.x | Tailwind Vite plugin | Official Vite integration for Tailwind v4. |
| **shadcn/ui** | 4.3.x (CLI) | Component library | Not a traditional npm package -- CLI copies component source code into your project. Built on Radix UI primitives + Tailwind. Full control over components. Ideal for dashboard UI (tables, dialogs, cards, dropdowns). |
| **Recharts** | 3.8.x | Chart library | Built on React + D3. Declarative components for PieChart, LineChart, BarChart -- exactly what the dashboard needs (allocation pie charts, net worth trend lines, bar charts by category). React 19 compatible (peerDeps: `^16 \|\| ^17 \|\| ^18 \|\| ^19`). |
| **Radix UI** | latest | Headless UI primitives | Underlying shadcn/ui components. Accessible, unstyled, composable. |
| **lucide-react** | 1.8.x | Icon library | Clean, consistent icon set. Used by shadcn/ui by default. |
| **react-router** | 7.14.x | Client-side routing | Current standard. This app has few routes (dashboard, settings, assets) so file-based routing from Next.js is unnecessary. React Router v7 is lean. |

### Frontend Forms & Validation

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **react-hook-form** | 7.72.x | Form state management | For manual asset entry forms, allocation target forms. Minimal re-renders, built-in validation. |
| **@hookform/resolvers** | 5.2.x | Zod resolver for RHF | Connects Zod schemas to react-hook-form validation. |
| **zod** | 4.3.x | Schema validation | Shared validation schemas between frontend forms and API responses. TypeScript-first, type inference. Also used on backend for config validation. |

### Backend Utilities

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **class-validator** | 0.15.x | DTO validation (NestJS native) | NestJS uses class-validator decorators for request validation pipes (`@IsString()`, `@IsNumber()`, etc.). Standard pattern. |
| **class-transformer** | 0.5.x | DTO transformation | Paired with class-validator. Transforms plain objects to class instances for validation. |
| **axios** | 1.15.x | HTTP client | For calling market data APIs (yahoo-finance2 uses it internally, but also useful for any additional HTTP calls). |
| **date-fns** | 4.1.x | Date manipulation | Tree-shakeable date library. For formatting snapshot dates, computing date ranges for trend charts, time-series grouping. Much lighter than moment.js (deprecated). |

### Logging

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **NestJS built-in Logger** | built-in | Application logging | NestJS includes a configurable logger. For a single-user app, this is sufficient. No need for Pino/Winston complexity. Upgrade to Pino if log volume demands it later. |

### Development Tools

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **@nestjs/cli** | 11.12.x | NestJS scaffolding | `nest g module`, `nest g service`, etc. for code generation. |
| **ESLint** | 10.2.x | Linting | Standard. Flat config format is now default in ESLint 10. |
| **Prettier** | 3.8.x | Code formatting | Standard. Run via IDE integration and pre-commit hooks. |
| **Vitest** | 4.1.x | Frontend testing | Vite-native test runner. Faster than Jest for frontend. Same API as Jest. |
| **Jest** | 30.x | Backend testing | NestJS default test runner. `@nestjs/testing` is built for Jest. |
| **ts-jest** | 29.x | TypeScript Jest transform | Required for Jest to handle TypeScript in NestJS tests. |
| **@testing-library/react** | 16.3.x | React component testing | Standard for testing React components by querying rendered output. |
| **MSW** | 2.13.x | API mocking | Mock Service Worker for intercepting API calls in frontend tests. |

### Infrastructure

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **Docker** | latest | Containerization | User-specified constraint. Separate Dockerfiles for backend, frontend, database. |
| **docker-compose** | v2+ | Multi-container orchestration | User-specified. Manages backend + frontend + PostgreSQL containers together. |
| **Node.js** | 22 LTS | Runtime | Current LTS. Both NestJS and React 19 support it. (v24 is current but not LTS yet -- pin to 22 LTS for stability.) |

## Alternatives Considered

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| ORM | **TypeORM** 0.3.x | Prisma 7.7.x | Prisma has superior DX (auto-generated types, Prisma Studio, cleaner migrations). However, TypeORM has deeper NestJS integration via `@nestjs/typeorm`, supports both Active Record and Data Mapper patterns, and the NestJS docs use TypeORM examples throughout. For a solo-developer project with NestJS, TypeORM's tighter integration outweighs Prisma's DX advantages. Prisma also adds a separate schema language and generation step that adds complexity. |
| State Management | **Zustand** 5.x | Redux Toolkit | Redux is overkill for a dashboard app. Zustand gives global state with zero boilerplate. No actions, reducers, or providers. For this app's state (portfolio data, UI preferences), Zustand is the right size. |
| Data Fetching | **@tanstack/react-query** | SWR | React Query has more features (prefetching, infinite queries, mutation handling, devtools). For dashboard data that needs background refresh (CCXT sync status), React Query is the better choice. |
| Charting | **Recharts** 3.x | D3 raw, Victory, Nivo | D3 is too low-level for dashboard charts. Victory is React-native focused. Nivo is good but Recharts has the best React integration with declarative components and full React 19 support. For pie/line/bar charts, Recharts is the right abstraction level. |
| Build Tool | **Vite** 8.x | Webpack, Turbopack | Webpack is legacy. Turbopack is Next.js-specific. Vite is the standard for React SPAs. |
| CSS | **Tailwind** 4.x | CSS Modules, Styled Components | Tailwind v4 with Vite plugin is zero-config. For a dashboard with many similar-styled elements (cards, tables, charts), utility classes are faster than writing component-scoped CSS. |
| Market Data | **yahoo-finance2** | Alpha Vantage, Polygon.io, Finnhub | Alpha Vantage has rate limits (5 req/min free). Polygon.io and Finnhub require API keys with limited free tiers. yahoo-finance2 is free, no API key, no rate limiting for personal use, covers stocks/ETFs/ETPs globally. Trade-off: no SLA, community-maintained. Acceptable for a personal tool. |
| Forms | **react-hook-form + zod** | Formik | Formik is effectively unmaintained. RHF + Zod is the current standard. |
| Routing | **react-router** 7.x | TanStack Router | TanStack Router is newer with type-safe routing, but adds complexity. React Router v7 is proven and this app has very few routes. |

## What NOT to Use

| Technology | Why Avoid | What to Use Instead |
|------------|-----------|-------------------|
| **Next.js** | This is a single-user SPA with a separate NestJS backend. Next.js adds SSR, routing, and deployment complexity that provides zero benefit. The backend already exists as NestJS. | Vite + React SPA |
| **Create React App** | Deprecated. No longer maintained. | Vite |
| **moment.js** | Deprecated by its own maintainers. Mutable dates, massive bundle size. | date-fns |
| **Redux / Redux Toolkit** | Overkill for this scope. Boilerplate-heavy for a dashboard with a handful of state slices. | Zustand |
| **Express raw (without NestJS)** | Loses modular architecture, DI, decorators, testing utilities, scheduled tasks, config management. | NestJS |
| **MongoDB** | Portfolio data is relational (assets belong to categories, snapshots reference assets, targets are per-category). PostgreSQL is the right fit. | PostgreSQL |
| **Socket.io / WebSockets** | Not needed for a single-user app. Dashboard refreshes via polling or manual refresh. CCXT sync is server-side cron, not real-time pushed to client. | HTTP REST + React Query background refetch |
| **GraphQL** | Overkill for this API surface. The app has ~10 endpoints. REST is simpler and NestJS controllers are natural REST handlers. | REST API |
| **Auth0 / Passport.js** | Single user, no auth required per project spec. If auth is ever needed, add a simple API key middleware. | No auth (per spec) |
| **Firebase / Supabase** | Adds external dependency. All data is local, containerized. Self-hosted PostgreSQL with TypeORM keeps everything self-contained. | PostgreSQL + TypeORM |
| **Bootstrap / MUI** | Heavy, opinionated, hard to customize for dashboard layouts. MUI's JSS-based styling conflicts with Tailwind. | Tailwind + shadcn/ui |
| **Jest for frontend** | Jest with Vite requires configuration bridges (vitest-fixture). Vitest is the native Vite test runner with same API. | Vitest for frontend, Jest for backend |
| **node-cron directly** | Bypasses NestJS's scheduling module. `@nestjs/schedule` provides decorators and `SchedulerRegistry` for dynamic job management. | @nestjs/schedule |

## Version Compatibility Matrix

| Component | Version | Compatible With | Notes |
|-----------|---------|----------------|-------|
| Node.js | 22 LTS | NestJS 11, React 19, TypeORM 0.3, CCXT 4.x | Pin to 22 LTS in Dockerfile |
| TypeScript | 6.0.x | NestJS 11, React 19, TypeORM 0.3 | Shared across frontend/backend |
| NestJS | 11.1.x | @nestjs/* 11.x packages | All @nestjs packages must match major version |
| React | 19.2.x | Recharts 3.x, shadcn/ui, react-router 7.x | React 19 stable since Dec 2024 |
| TypeORM | 0.3.x | @nestjs/typeorm 11.x, pg 8.x | v0.3 is latest stable (0.4 not yet released) |
| CCXT | 4.5.x | Node 22, TypeScript 6 | v4 is current major |
| Tailwind | 4.2.x | Vite 8.x via @tailwindcss/vite | v4 uses CSS-first config (no tailwind.config.js) |
| Vite | 8.0.x | React 19 via @vitejs/plugin-react 6.x | Current major |
| React Query | 5.99.x | React 19 | TanStack Query v5 is current |
| Zustand | 5.0.x | React 19 | v5 with React 19 support |

## Installation

```bash
# === BACKEND (NestJS) ===
# Core NestJS
npm install @nestjs/core@^11 @nestjs/common@^11 @nestjs/platform-express@^11 @nestjs/cli@^11
# NestJS modules
npm install @nestjs/config@^4 @nestjs/schedule@^6 @nestjs/typeorm@^11 @nestjs/testing@^11
# Database
npm install typeorm@^0.3 pg@^8
# Crypto exchange
npm install ccxt@^4
# Market data
npm install yahoo-finance2@^3
# Validation
npm install class-validator@^0.15 class-transformer@^0.5 zod@^4
# HTTP & utilities
npm install axios@^1 date-fns@^4

# Backend dev dependencies
npm install -D typescript@^6 @types/node@^22 ts-jest@^29 jest@^30 @nestjs/cli@^11 eslint@^10 prettier@^3

# === FRONTEND (React + Vite) ===
# Core React
npm install react@^19 react-dom@^19
# Build tool
npm install -D vite@^8 @vitejs/plugin-react@^6
# Routing
npm install react-router@^7
# State management
npm install zustand@^5 @tanstack/react-query@^5
# Styling
npm install -D tailwindcss@^4 @tailwindcss/vite@^4
# UI components (shadcn via CLI, not npm)
npx shadcn@latest init
# Charts
npm install recharts@^3 react-is@^19
# Forms
npm install react-hook-form@^7 @hookform/resolvers@^5 zod@^4
# Icons & utilities
npm install lucide-react@^1 date-fns@^4

# Frontend dev dependencies
npm install -D typescript@^6 @types/react@^19 @types/react-dom@^19 vitest@^4 @testing-library/react@^16 @testing-library/jest-dom@^6 msw@^2 eslint@^10 prettier@^3
```

## Sources

- npm registry (verified 2026-04-20): All version numbers checked via `npm view <package> version`
- Context7 NestJS docs (`/nestjs/docs.nestjs.com`): TypeORM integration, scheduling module
- Context7 TypeORM docs (`/typeorm/typeorm`): PostgreSQL setup, migrations, DataSource API
- Context7 CCXT docs (`/ccxt/ccxt`): JavaScript fetchBalance, exchange initialization
- Context7 React docs (`/facebook/react`): React 19 features
- Recharts npm page: React 19 peer dependency confirmed (`^16 || ^17 || ^18 || ^19`)
- yahoo-finance2 npm page: Server-side only, TypeScript support, beta status noted

## Confidence Assessment

| Area | Confidence | Reason |
|------|------------|--------|
| NestJS ecosystem | HIGH | Versions verified on npm, docs confirmed via Context7 |
| React + Vite stack | HIGH | Current standard, versions verified |
| TypeORM over Prisma | MEDIUM | Judgment call; Prisma has valid strengths but TypeORM's NestJS integration is decisive |
| CCXT | HIGH | User-specified, version verified |
| yahoo-finance2 | MEDIUM | Community package with beta label; no better free alternative exists |
| Tailwind + shadcn/ui | HIGH | Current standard for React dashboards |
| Recharts | HIGH | React 19 compatible, widely used for dashboard charts |
| Zustand over Redux | HIGH | Clear fit for single-user dashboard scope |
