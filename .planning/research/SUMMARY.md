# Project Research Summary

**Project:** Finance Manager -- Personal Portfolio Management
**Domain:** Personal finance / portfolio tracker (single-user, self-hosted)
**Researched:** 2026-04-20
**Confidence:** HIGH

## Executive Summary

This is a single-user personal finance dashboard that aggregates crypto exchange balances (via CCXT), manual asset entries (bank accounts, stocks, ETFs, ETPs), and auto-fetched market prices into a unified net worth view denominated in EUR. The domain is well-established: tools like Ghostfolio and Sharesight demonstrate mature patterns for portfolio tracking, but none offer multi-level rebalancing suggestions, which is this project's primary differentiator.

The recommended approach is a three-tier architecture (React SPA + NestJS REST API + PostgreSQL) containerized with Docker. The backend is organized into six domain modules (Assets, Exchanges, Sync, MarketData, Portfolio, Dashboard) following NestJS's module-per-domain convention. Data flows in one direction: external APIs are polled on schedule, data is normalized and stored, and the dashboard reads only from the database. The frontend uses React Query for server state, Zustand for client state, Recharts for visualizations, and shadcn/ui with Tailwind for the component layer.

The principal risks are (1) floating-point precision in monetary calculations -- mitigated by using PostgreSQL DECIMAL(20,8) columns and a decimal library, (2) CCXT error handling -- mitigated by distinguishing NetworkError from ExchangeError from day one, and (3) API key security -- mitigated by AES-256 encryption at rest. These must be addressed in the foundation phase; retrofitting them is disproportionately expensive.

## Key Findings

### Recommended Stack

The stack is a NestJS 11 + React 19 + PostgreSQL 17 combination, chosen to match the user's constraints and the project's needs. All versions were verified against the npm registry.

**Core technologies:**
- **NestJS 11.1.x** -- backend framework with modular architecture, DI, decorators, and native scheduling support
- **React 19.2.x + Vite 8.x** -- SPA frontend with fast HMR and native ESM builds
- **PostgreSQL 17.x + TypeORM 0.3.x** -- relational storage for entity-heavy domain; TypeORM chosen over Prisma for tighter NestJS integration
- **CCXT 4.5.x** -- unified crypto exchange API across 100+ exchanges; user-specified
- **yahoo-finance2 3.14.x** -- free stock/ETF/ETP price data and FX rates; community-maintained but the only viable free option
- **Recharts 3.8.x** -- declarative React chart library for pie charts, line charts, and bar charts; React 19 compatible
- **shadcn/ui + Tailwind 4.x** -- copy-into-project component library with utility-first CSS; ideal for dashboard layouts
- **Zustand 5.x + React Query 5.x** -- minimal global state plus server-state caching with background refetch
- **react-hook-form + Zod 4.x** -- form management with shared validation schemas between frontend and backend

Full stack details with version matrix and installation commands: see [STACK.md](./STACK.md)

### Expected Features

Features are organized by dependency chain. The data layer (asset entry, exchange sync, price fetching) must be complete before aggregation, which must be complete before analytics.

**Must have (table stakes):**
- Total net worth dashboard -- the core value proposition
- Multi-currency conversion to EUR -- non-negotiable for a multi-asset portfolio
- CCXT crypto balance import -- primary automated data source
- Manual asset entry (bank accounts, stocks, ETFs, ETPs) -- not everything has an API
- Auto-fetched market prices for tradable assets -- stale prices make the dashboard useless
- Portfolio allocation pie chart -- every portfolio tracker has this
- Historical net worth trend line -- requires daily snapshot persistence
- Auto-polling data sync on schedule with manual refresh -- both automated and on-demand

**Should have (differentiators):**
- Multi-level rebalancing suggestions (macro, liquidity, intra-category) -- genuinely uncommon in self-hosted tools
- Emergency fund as protected allocation -- first-class constraint with visual status
- Trading Bot tagged accounts -- separates active trading from passive holdings
- Allocation drift indicators -- visual cue when portfolio has drifted from targets
- Per-asset cost basis tracking with unrealized P&L

**Defer (v2+):**
- Trade execution, mobile native app, push notifications, tax reporting, social features, news feed, real-time streaming

Feature dependency graph and complexity matrix: see [FEATURES.md](./FEATURES.md)

### Architecture Approach

The system uses six NestJS domain modules with strict one-directional data flow: external APIs are polled on schedule by the Sync and MarketData modules, data is persisted in PostgreSQL, and the Dashboard module composes a single aggregated response for the frontend. The frontend makes one request per page load, not a waterfall of API calls. Snapshots are append-only. API keys are encrypted at rest.

**Major components:**
1. **Assets module** -- CRUD for manual assets (bank accounts, stocks, ETFs, ETPs); the foundational data entity
2. **Exchanges + Sync modules** -- CCXT connection management, encrypted credential storage, scheduled balance fetching, sync logging
3. **MarketData module** -- Price cache with TTL, FX rate management via yahoo-finance2; reduces external API calls by ~99%
4. **Portfolio module** -- Net worth calculation, allocation breakdowns, target allocation hierarchy, rebalancing engine, historical snapshots
5. **Dashboard module** -- Read-only aggregation layer with zero entities; composes data from other services into a single response DTO
6. **React SPA** -- Five page areas (Dashboard, Assets, Exchanges, Allocation, History) consuming backend REST APIs via React Query

Database entity model, API surface, and data flow diagrams: see [ARCHITECTURE.md](./ARCHITECTURE.md)

### Critical Pitfalls

1. **Storing monetary values as JavaScript floats** -- Use PostgreSQL DECIMAL(20,8) and a decimal library. JavaScript `0.1 + 0.2 !== 0.3` silently corrupts portfolio totals. Must be decided in Phase 1; schema migration later is painful.
2. **Not distinguishing CCXT NetworkError from ExchangeError** -- Network errors are retryable; exchange errors (bad API key) are not. A single catch block causes either retry storms or silently dropped syncs. Must be correct from the first exchange connection.
3. **Storing API keys unencrypted at rest** -- AES-256 encryption using Node.js built-in crypto module. Master key from environment variable, not the database. Retrofitting encryption after launch requires migrating all stored credentials.
4. **Single currency assumption in data model** -- Every monetary value stores both amount and currency. Base currency (EUR) is for display, not for discarding source currency. Without source currency, FX gains cannot be separated from asset gains.
5. **Rebalancing math that ignores drift tolerance** -- Suggestions for sub-1% deviations are noise. Define tolerance bands (e.g., 5% macro, 3% liquidity), minimum rebalance amounts, and cache calculations. A technically correct but practically useless rebalancer erodes trust.

Full pitfall catalog with detection methods and phase mappings: see [PITFALLS.md](./PITFALLS.md)

## Implications for Roadmap

Based on combined research, the following phase structure is recommended. The ordering follows the dependency chain identified in both the feature dependency graph and the module dependency graph.

### Phase 1: Foundation and Data Model

**Rationale:** Everything depends on the database schema, Docker setup, and project structure. Getting monetary precision, multi-currency columns, and snapshot idempotency constraints right here prevents costly migrations later. Docker networking should lock down the backend from the start.

**Delivers:** Running NestJS backend, PostgreSQL, Docker Compose, React SPA shell, TypeORM migrations, encryption utilities, common DTOs and error filters, manual asset CRUD.

**Addresses:** Manual asset entry (table stakes), responsive web layout, asset categorization

**Avoids:** CP-1 (monetary floats), CP-4 (single currency), TDP-3 (snapshot idempotency), PT-1 (N+1 queries via good data model), SM-2 (no CORS restrictions)

### Phase 2: External Integrations

**Rationale:** Crypto exchange balances and market prices are the two external data sources. Both must be working before any aggregation or analytics. CCXT integration is the highest-risk component; it should be built with proper error handling, instance reuse, and credential encryption from the start. Market data and FX rates from yahoo-finance2 should be built alongside with caching.

**Delivers:** Exchange connection CRUD with encrypted keys, CCXT balance fetching, scheduled sync with per-exchange intervals, market price fetching with TTL cache, FX rate management, sync status tracking, manual refresh endpoints, frontend exchange configuration and asset list pages.

**Addresses:** CCXT crypto import, auto-fetched market prices, multi-currency conversion, auto-polling sync, manual refresh, trading bot tagging

**Avoids:** CP-2 (CCXT error handling), CP-3 (unencrypted keys), TDP-1 (CCXT instance reuse), TDP-2 (uniform polling), IG-1 through IG-5 (all integration gotchas), SM-1 (API keys in logs), SM-3 (credentials in .env)

### Phase 3: Dashboard and Analytics

**Rationale:** The value layer. With all data flowing in from Phase 2 and manual assets from Phase 1, the portfolio module can compute net worth, allocations, and rebalancing suggestions using real data from day one. Dashboard caching ensures sub-second load times. Historical tracking adds the time dimension.

**Delivers:** Net worth dashboard with total and breakdown, allocation pie chart, historical net worth trend line, emergency fund status card, allocation target configuration, multi-level rebalancing suggestions, drift indicators, per-asset P&L, full frontend dashboard with charts.

**Addresses:** Net worth dashboard, allocation pie chart, historical trend line, emergency fund status, multi-level rebalancing, drift indicators, cost basis and P&L

**Avoids:** CP-5 (rebalancing without drift tolerance), PT-2 (recalculating on every load), PT-3 (unbounded historical queries), UX-1 through UX-3 (formatting, stale data indicators, emergency fund visibility)

### Phase Ordering Rationale

- **Phase 1 before Phase 2:** The data model (especially DECIMAL columns, currency fields, and encryption utilities) must exist before any external data is persisted. Retrofitting precision or multi-currency support into an existing database is a high-risk migration.
- **Phase 2 before Phase 3:** Portfolio analytics depends on accurate, current data from both manual assets and synced exchanges. Building analytics against real data validates correctness immediately.
- **Phase 3 is last because it is pure computation:** No new external integrations, no new data sources. It reads from what Phases 1 and 2 built. This makes it the safest phase to iterate on.

### Research Flags

Phases likely needing deeper research during planning:

- **Phase 2 (CCXT integration):** Highest-risk phase. Needs research on per-exchange balance quirks, sandbox testing setup, rate limit tuning, and the exact CCXT error hierarchy implementation. The sync scheduler design (per-exchange intervals, stagger, retry logic) should be researched before implementation.
- **Phase 3 (Rebalancing engine):** The multi-level rebalancing algorithm (macro -> liquidity -> intra-category with target hierarchies) is not a standard library pattern. Drift tolerance values and the suggestion ranking algorithm need design work. Consider a `/gsd-research-phase` call for the rebalancing math.

Phases with standard patterns (skip research-phase):

- **Phase 1 (Foundation):** NestJS scaffolding, TypeORM setup, Docker Compose, and basic CRUD are well-documented with established patterns. No research needed.
- **Phase 2 (Market data sub-component):** yahoo-finance2 usage is straightforward. Price cache with TTL is a standard pattern. FX rate fetching is documented.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All versions verified against npm registry. NestJS, React, TypeORM, CCXT docs confirmed via Context7. |
| Features | HIGH | Feature dependency chain is clear. Competitor analysis (Ghostfolio, Sharesight) confirms table stakes vs. differentiators. Multi-level rebalancing is validated as a genuine differentiator. |
| Architecture | HIGH | Module boundaries are clean. Data flow is unidirectional. NestJS patterns are well-documented. Build order follows dependency chain logically. |
| Pitfalls | HIGH | CCXT error hierarchy, TypeORM decimal handling, and yahoo-finance2 rate limiting sourced from Context7 library docs. Monetary precision is a known universal pitfall. Rebalancing drift tolerance is domain expertise but values should be validated. |

**Overall confidence:** HIGH

### Gaps to Address

- **yahoo-finance2 reliability:** Community-maintained, no SLA. If Yahoo Finance changes their unofficial API, this library breaks. Mitigation: implement aggressive caching and a "stale data" indicator. Have a fallback plan (e.g., manual price entry) if the library becomes unusable.
- **Rebalancing tolerance values:** The research recommends 5% macro, 3% liquidity, 5% intra-category drift bands, but these are domain-heuristic values. Validate with the user during Phase 3 planning before hardcoding.
- **CCXT per-exchange balance quirks:** Research identifies that not all exchanges report `free`/`used`/`total` consistently, but the specific quirks of the user's exchanges (Binance, Kraken, etc.) should be tested during Phase 2 implementation.
- **TypeORM DECIMAL handling:** PostgreSQL returns DECIMAL columns as strings. The exact integration pattern (keep as string, parse to decimal.js, or use a TypeORM transformer) needs a concrete decision during Phase 1 implementation.

## Sources

### Primary (HIGH confidence)

- Context7 NestJS docs (`/nestjs/docs.nestjs.com`) -- modules, TypeORM integration, scheduling, testing
- Context7 TypeORM docs (`/typeorm/typeorm`) -- PostgreSQL setup, migrations, DECIMAL types, DataSource API
- Context7 CCXT docs (`/ccxt/ccxt`) -- fetchBalance, error hierarchy, loadMarkets, rate limiting, balance structure
- Context7 yahoo-finance2 docs (`/gadicc/yahoo-finance2`) -- quote, historical, forex pairs, error types, concurrency
- Context7 React docs (`/facebook/react`) -- React 19 features
- npm registry (verified 2026-04-20) -- all version numbers confirmed

### Secondary (MEDIUM confidence)

- Ghostfolio GitHub README (github.com/ghostfolio/ghostfolio) -- feature inventory, tech stack comparison
- Sharesight features page (sharesight.com/features) -- commercial competitor feature list
- CCXT library on npmjs.com -- package metadata, usage patterns
- Recharts npm page -- React 19 peer dependency confirmation

### Tertiary (LOW confidence)

- Portfolio rebalancing theory -- drift tolerance values are domain heuristics; validate with user
- Multi-currency FX patterns -- standard financial application patterns; not sourced from a specific library

---
*Research completed: 2026-04-20*
*Ready for roadmap: yes*
