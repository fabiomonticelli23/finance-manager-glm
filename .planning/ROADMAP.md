# Roadmap: Finance Manager

## Overview

Build a personal portfolio dashboard that starts with a solid data model and manual asset entry, then adds automated data ingestion from crypto exchanges and market price APIs, then delivers a visual dashboard showing total net worth and allocation, and finally adds the multi-level rebalancing engine that provides actionable portfolio guidance. Each phase delivers a coherent, verifiable capability that the next phase builds on.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3, 4): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation and Manual Assets** - Project scaffolding, data model with monetary precision, Docker setup, and full CRUD for manual asset management
- [ ] **Phase 2: External Integrations and Sync** - CCXT exchange connections, market price fetching, FX conversion, and automated/manual data sync
- [ ] **Phase 3: Dashboard and Portfolio Analytics** - Total net worth display, allocation charts, historical trend tracking, and drift indicators
- [ ] **Phase 4: Multi-Level Rebalancing Engine** - Target allocation configuration at all levels, rebalancing suggestions, and emergency fund protection

## Phase Details

### Phase 1: Foundation and Manual Assets
**Goal**: Users can manually manage all their non-exchange assets in a running, containerized application
**Depends on**: Nothing (first phase)
**Requirements**: DATA-01, DATA-02, DATA-07
**Success Criteria** (what must be TRUE):
  1. User can add a manual asset (bank account, stock, ETF, ETP) with name, type, quantity, currency, and institution, and see it persisted in the application
  2. User can edit any existing manual asset's details and see the changes reflected immediately
  3. User can delete a manual asset and confirm it is removed from the application
  4. User can assign each asset to a macro category (Crypto, Stocks, Liquidity, Trading Bots) and filter assets by category
  5. The entire application (backend, frontend, database) starts with a single `docker compose up` command
**Plans**: 4 plans

Plans:
- [ ] 01-01-PLAN.md -- Backend scaffold: NestJS + TypeORM + Asset CRUD API with unit tests
- [ ] 01-02-PLAN.md -- Frontend scaffold: Vite + React + Tailwind v4 + shadcn/ui app shell with sidebar navigation
- [ ] 01-03-PLAN.md -- Frontend CRUD UI: asset list with category tabs, dynamic form, delete dialog
- [ ] 01-04-PLAN.md -- Docker Compose, Dockerfiles, initial migration, full-stack integration

**UI hint**: yes

### Phase 2: External Integrations and Sync
**Goal**: Users can connect crypto exchanges for automatic balance import and the system keeps all market prices and FX rates current
**Depends on**: Phase 1
**Requirements**: DATA-03, DATA-04, DATA-05, DATA-06, SYNC-01, SYNC-02, TBOT-01
**Success Criteria** (what must be TRUE):
  1. User can add a crypto exchange connection by providing exchange name and API keys, and see the imported balances appear as assets
  2. User can disconnect an exchange connection and see its synced balances removed
  3. User can tag a CCXT-connected account as "Trading Bot" and see it grouped under a separate Trading Bots macro category
  4. System automatically fetches current market prices for all stocks, ETFs, and ETPs and converts all non-EUR values to EUR
  5. System auto-polls exchanges and market prices on a configurable schedule without user intervention
  6. User can trigger an immediate data refresh and see updated balances and prices
**Plans**: TBD

**UI hint**: yes

### Phase 3: Dashboard and Portfolio Analytics
**Goal**: Users can see their complete financial picture on a single dashboard with charts showing allocation and historical trends
**Depends on**: Phase 2
**Requirements**: DASH-01, DASH-02, DASH-03, DASH-04
**Success Criteria** (what must be TRUE):
  1. User can see total net worth across all assets (manual + exchange-synced) displayed in EUR on a single dashboard page
  2. User can see portfolio allocation by macro category in a pie chart that reflects all asset types
  3. User can see historical net worth over time in a line chart showing the portfolio's trajectory
  4. User can see visual drift indicators when any macro category allocation deviates from target percentages
**Plans**: TBD

**UI hint**: yes

### Phase 4: Multi-Level Rebalancing Engine
**Goal**: Users receive actionable rebalancing guidance at every portfolio level with emergency fund protection
**Depends on**: Phase 3
**Requirements**: REBAL-01, REBAL-02, REBAL-03, REBAL-04, REBAL-05, REBAL-06, REBAL-07, REBAL-08
**Success Criteria** (what must be TRUE):
  1. User can set target allocation percentages at macro level and see concrete rebalancing suggestions (e.g., "move X EUR from Stocks to Crypto")
  2. User can set target allocation within the Liquidity category and see liquidity-level rebalancing suggestions
  3. User can set target allocation within each macro category and see intra-category suggestions (e.g., BTC vs ETH allocation within Crypto)
  4. User can set an emergency fund target value or percentage and the system excludes it from rebalancing suggestions
  5. User can see emergency fund status (underfunded, on target, overfunded) prominently on the dashboard
**Plans**: TBD

**UI hint**: yes

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3 -> 4

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation and Manual Assets | 0/4 | Not started | - |
| 2. External Integrations and Sync | 0/? | Not started | - |
| 3. Dashboard and Portfolio Analytics | 0/? | Not started | - |
| 4. Multi-Level Rebalancing Engine | 0/? | Not started | - |
