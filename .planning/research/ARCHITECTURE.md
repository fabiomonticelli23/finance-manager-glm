# Architecture Patterns

**Domain:** Personal finance portfolio management (single-user)
**Researched:** 2026-04-20
**Stack:** NestJS + React + PostgreSQL, containerized with Docker

---

## System Overview

This is a single-user personal finance dashboard that aggregates data from external sources (crypto exchanges, market price APIs) and manual user input into a unified net worth view. The architecture follows a strict three-tier separation: React SPA frontend, NestJS REST API backend, PostgreSQL database. No authentication layer is needed (single user, local deployment).

```
                    +-------------------+
                    |   React SPA       |
                    |   (Dashboard UI)  |
                    +--------+----------+
                             |
                        REST API (JSON)
                             |
                    +--------v----------+
                    |   NestJS API      |
                    |   (Modules)       |
                    |                   |
                    |  +-------------+  |
                    |  | CCXT Sync   |  |<--- Scheduled polling
                    |  | Service     |  |     (cron jobs)
                    |  +-------------+  |
                    |         |         |
                    |  +-------------+  |
                    |  | Market Data |  |<--- External price APIs
                    |  | Service     |  |
                    |  +-------------+  |
                    |         |         |
                    |  +-------------+  |
                    |  | Aggregation |  |
                    |  | Service     |  |
                    |  +-------------+  |
                    +--------+----------+
                             |
                        TypeORM
                             |
                    +--------v----------+
                    |   PostgreSQL      |
                    |   (Entities)      |
                    +-------------------+
```

---

## Component Responsibilities

### Backend: NestJS Modules

The backend is organized into domain modules. Each module owns its entities, services, controllers, and DTOs. Modules communicate through well-defined service interfaces, never by sharing repositories directly.

| Module | Responsibility | Key Entities | Communicates With |
|--------|---------------|--------------|-------------------|
| **Assets** | CRUD for all user-held assets (manual entry). Manages bank accounts, stocks, ETFs, ETPs, emergency fund records. | `Asset`, `AssetType` enum | MarketData (for price enrichment), Portfolio (read) |
| **Exchanges** | Stores CCXT connection configs (exchange name, API keys). Manages exchange lifecycle. | `ExchangeConnection` | Sync (provides config to CCXT), Assets (creates synced assets) |
| **Sync** | Runs CCXT balance fetches on schedule and on demand. Persists fetched balances as assets. | `SyncLog`, `SyncStatus` | Exchanges (reads config), Assets (writes fetched balances) |
| **MarketData** | Fetches and caches current market prices for stocks, ETFs, ETPs. Handles currency conversion rates. | `PriceCache`, `FxRate` | Assets (provides prices), Portfolio (provides conversion rates) |
| **Portfolio** | Computes total net worth, allocation breakdowns, drift from targets. Generates rebalancing suggestions. | `AllocationTarget`, `NetWorthSnapshot` | Assets (reads all), MarketData (reads FX rates) |
| **Dashboard** | Aggregation controller that composes data from multiple modules into dashboard-specific DTOs. | None (read-only aggregation) | Portfolio, Assets, MarketData |

### Module Dependency Graph

```
Dashboard --> Portfolio --> Assets --> MarketData
                         --> Exchanges --> Sync --> Assets
Portfolio --> MarketData
```

Build order must respect this dependency chain. Portfolio depends on both Assets and MarketData, so both must be built first. Sync depends on Exchanges and writes to Assets.

### Frontend: React SPA

The frontend is a single-page application organized by feature areas that mirror backend API endpoints.

| Component Area | Responsibility | API Consumer |
|---------------|---------------|--------------|
| **Dashboard Page** | Net worth card, allocation pie chart, drift indicators, emergency fund status | Dashboard API |
| **Assets Page** | List/add/edit manual assets (bank accounts, stocks, ETFs) | Assets API |
| **Exchanges Page** | Configure CCXT connections (add/edit API keys), view sync status, trigger manual sync | Exchanges API + Sync API |
| **Allocation Page** | Set target allocations at macro/liquidity/intra-category levels, view current vs target | Portfolio API |
| **History Page** | Net worth trend line chart over time | Portfolio API (snapshots) |
| **Shared Layout** | Navigation, EUR currency display, consistent styling | None |

### Database: PostgreSQL via TypeORM

Single database, single schema. TypeORM entities with migrations (not synchronize in production). Key design decisions:

- **Encrypted API keys at rest.** Exchange API keys are encrypted before storage and decrypted only in the Sync service at runtime. Use a dedicated encryption utility with an environment-variable-driven key.
- **Soft deletes on assets.** Users may want to hide old assets without losing historical snapshots.
- **Timestamps on everything.** `createdAt` and `updatedAt` on all entities for audit trail.
- **Snapshots are append-only.** Net worth snapshots are never updated or deleted once created.

---

## Project Structure

```
finance-manager-glm/
  docker-compose.yml
  backend/
    Dockerfile
    src/
      app.module.ts
      main.ts
      config/
        config.module.ts
        config.service.ts          # Wraps @nestjs/config
      common/
        decorators/
        interceptors/
        filters/
        dto/
      modules/
        assets/
          assets.module.ts
          assets.controller.ts
          assets.service.ts
          dto/
          entities/
            asset.entity.ts
            asset-type.enum.ts
        exchanges/
          exchanges.module.ts
          exchanges.controller.ts
          exchanges.service.ts
          dto/
          entities/
            exchange-connection.entity.ts
        sync/
          sync.module.ts
          sync.controller.ts
          sync.service.ts          # CCXT integration
          sync.scheduler.ts        # @Cron decorated methods
          dto/
          entities/
            sync-log.entity.ts
        market-data/
          market-data.module.ts
          market-data.controller.ts
          market-data.service.ts
          dto/
          entities/
            price-cache.entity.ts
            fx-rate.entity.ts
        portfolio/
          portfolio.module.ts
          portfolio.controller.ts
          portfolio.service.ts
          dto/
          entities/
            allocation-target.entity.ts
            net-worth-snapshot.entity.ts
        dashboard/
          dashboard.module.ts
          dashboard.controller.ts
          dashboard.service.ts
          dto/
    migrations/
  frontend/
    Dockerfile
    src/
      App.tsx
      api/
        assets.ts
        exchanges.ts
        sync.ts
        portfolio.ts
        dashboard.ts
      components/
        layout/
          AppLayout.tsx
          Sidebar.tsx
        dashboard/
          NetWorthCard.tsx
          AllocationPieChart.tsx
          DriftIndicator.tsx
          EmergencyFundStatus.tsx
        assets/
          AssetList.tsx
          AssetForm.tsx
        exchanges/
          ExchangeList.tsx
          ExchangeForm.tsx
          SyncStatusBadge.tsx
        allocation/
          TargetAllocationForm.tsx
          AllocationComparison.tsx
        history/
          NetWorthChart.tsx
      hooks/
        useAssets.ts
        usePortfolio.ts
        useSync.ts
      types/
        api.ts
  .env.example
```

---

## Architectural Patterns

### Pattern 1: Module-per-Domain (NestJS)

**What:** Each business domain (assets, exchanges, sync, market-data, portfolio, dashboard) is a self-contained NestJS module with its own controller, service, entities, and DTOs.

**When:** Always in NestJS. This is the framework's core organizing principle.

**Why:** NestJS modules create explicit dependency boundaries. `imports` and `exports` make data flow visible. A module can only use what it explicitly imports. This prevents circular dependencies and keeps the dependency graph clean.

```
@Module({
  imports: [TypeOrmModule.forFeature([Asset])],
  controllers: [AssetsController],
  providers: [AssetsService],
  exports: [AssetsService],
})
export class AssetsModule {}
```

### Pattern 2: Aggregation Controller (Dashboard)

**What:** The Dashboard module does not own entities. Its service injects PortfolioService, AssetsService, and MarketDataService to compose a single response DTO with everything the dashboard needs.

**When:** Any read-heavy endpoint that needs data from multiple domains.

**Why:** Frontend should make one request per page load, not waterfall five API calls. The aggregation layer composes the response server-side.

```
@Injectable()
export class DashboardService {
  constructor(
    private portfolioService: PortfolioService,
    private assetsService: AssetsService,
  ) {}

  async getDashboard(): Promise<DashboardDto> {
    const [netWorth, allocation, drift] = await Promise.all([
      this.portfolioService.getCurrentNetWorth(),
      this.portfolioService.getAllocationBreakdown(),
      this.portfolioService.getDriftFromTargets(),
    ]);
    return { netWorth, allocation, drift };
  }
}
```

### Pattern 3: Scheduled Sync with Status Tracking

**What:** The Sync module uses `@nestjs/schedule` with `@Cron` decorators to periodically fetch CCXT balances. Each sync run creates a `SyncLog` entry recording start time, end time, status (success/partial/failed), and which exchanges were synced.

**When:** For any external data polling that needs auditability.

**Why:** Crypto exchange APIs are unreliable. Rate limits, downtime, and auth errors are common. The sync log lets the frontend show "last synced successfully at X" and flag failures. The scheduler itself wraps each sync in try-catch so one failing exchange does not block others.

```
@Injectable()
export class SyncScheduler {
  private readonly logger = new Logger(SyncScheduler.name);

  @Cron('*/15 * * * *')  // Every 15 minutes
  async handleScheduledSync() {
    const connections = await this.exchangesService.findAll();
    for (const conn of connections) {
      try {
        await this.syncService.syncExchange(conn);
      } catch (error) {
        this.logger.error(`Sync failed for ${conn.name}: ${error.message}`);
        // Continue with other exchanges
      }
    }
  }
}
```

### Pattern 4: Price Cache with TTL

**What:** Market prices and FX rates are fetched from external APIs and stored in a `PriceCache` / `FxRate` table with a `fetchedAt` timestamp. Services check cache freshness before calling external APIs. Stale entries are refreshed; fresh entries are returned directly.

**When:** Any external data that changes frequently but does not need to be real-time.

**Why:** Market data APIs have rate limits. Stock/ETF prices update every 15 minutes during market hours -- fetching every request is wasteful. A 15-minute TTL cache reduces API calls by ~99% for a single-user app while keeping prices reasonably current.

### Pattern 5: Append-Only Snapshots for History

**What:** A scheduled job (daily, or after each sync) computes total net worth and writes a `NetWorthSnapshot` row. Snapshots are never updated or deleted. The history page queries snapshots ordered by date.

**When:** Any time-series data for trend charts.

**Why:** Mutating historical records destroys audit value. Append-only is simple, queryable, and honest. If a user corrects an asset value, the next snapshot reflects the correction, but old snapshots remain as-is.

### Pattern 6: Encryption at Rest for API Keys

**What:** Exchange API keys and secrets are encrypted using AES-256 before storage. The encryption key comes from an environment variable. Only the Sync service decrypts them at runtime to instantiate CCXT clients.

**When:** Any credential storage in the database.

**Why:** Even though this is a single-user local app, storing API keys in plaintext in PostgreSQL is negligent. If the database volume is ever exposed (backup, Docker volume mount, dev machine access), encrypted keys limit damage.

---

## Data Flow

### Flow 1: CCXT Sync (Automated)

```
Cron Trigger (every 15 min)
    |
    v
SyncScheduler.handleScheduledSync()
    |
    v
ExchangesService.findAll() --> load all ExchangeConnection entities
    |
    v (for each connection)
SyncService.syncExchange(connection)
    |
    +-- Decrypt API keys from connection
    +-- Instantiate CCXT exchange client
    +-- Call exchange.fetchBalance()
    |
    v
For each non-zero balance:
    +-- Upsert Asset entity (crypto holding, linked to exchange)
    +-- Fetch current price via MarketDataService (for non-EUR conversion)
    |
    v
Write SyncLog entry (success/failure per exchange)
```

### Flow 2: Dashboard Load

```
Browser requests GET /api/dashboard
    |
    v
DashboardController.getDashboard()
    |
    v
DashboardService.getDashboard()
    |
    +-- PortfolioService.getCurrentNetWorth()
    |       +-- Sum all Assets (manual + synced) converted to EUR
    |       +-- MarketDataService provides FX rates
    |
    +-- PortfolioService.getAllocationBreakdown()
    |       +-- Group assets by macro category
    |       +-- Compute percentages
    |
    +-- PortfolioService.getDriftFromTargets()
    |       +-- Compare current allocation to AllocationTarget entities
    |       +-- Compute delta percentages
    |
    +-- PortfolioService.getEmergencyFundStatus()
            +-- Compare emergency fund value to target
            +-- Return underfunded/overfunded/ok status
    |
    v
Return DashboardDto (single JSON response)
```

### Flow 3: Manual Asset CRUD

```
Browser: POST /api/assets (create stock asset)
    |
    v
AssetsController.create(createAssetDto)
    |
    v
AssetsService.create()
    +-- Validate asset type, currency
    +-- Persist Asset entity
    +-- Trigger price fetch for symbol via MarketDataService
    |
    v
Return AssetDto (with current price populated)
```

### Flow 4: Rebalancing Suggestions

```
Browser: GET /api/portfolio/rebalancing
    |
    v
PortfolioController.getRebalancingSuggestions()
    |
    v
PortfolioService.calculateRebalancing()
    |
    +-- Load AllocationTarget entities (macro, liquidity, intra-category)
    +-- Load current allocation from Assets + MarketData
    +-- For each level:
    |       +-- Compute drift (current% - target%)
    |       +-- Compute EUR adjustment needed to close gap
    |       +-- Flag significant drift (e.g., > 5 percentage points)
    |
    v
Return RebalancingDto with suggested moves (advisory only, no execution)
```

---

## Database Entity Model

```
+---------------------+       +------------------------+
| ExchangeConnection  |       | Asset                  |
|---------------------|       |------------------------|
| id (PK, UUID)       |<--+   | id (PK, UUID)          |
| name (string)       |   |   | name (string)          |
| exchangeId (string) |   |   | type (enum)            |
| apiKey (encrypted)  |   |   | symbol (string, null)  |
| apiSecret (encrypt) |   |   | quantity (decimal)     |
| isTradingBot (bool) |   |   | currency (string)      |
| isActive (bool)     |   |  | exchangeId (FK, null) |
| createdAt           |   |   | category (enum)        |
| updatedAt           |   |  | currentValueEur (dec) |
+---------------------+   |   | purchasePrice (dec)    |
                          |   | isActive (bool)        |
                          |   | createdAt              |
                          |   | updatedAt              |
                          |   +------------------------+
                          |
                          |   +------------------------+
                          |   | AllocationTarget       |
                          |   |------------------------|
                          |   | id (PK, UUID)          |
                          +---| level (enum)            |  macro/liquidity/intra
                              | category (string)       |
                              | targetPercentage (dec)  |
                              | targetValueEur (dec)    |
                              | parentTargetId (FK)     |  self-ref for hierarchy
                              | createdAt               |
                              | updatedAt               |
                              +------------------------+

+---------------------+       +------------------------+
| PriceCache          |       | FxRate                 |
|---------------------|       |------------------------|
| id (PK, UUID)       |       | id (PK, UUID)          |
| symbol (string)     |       | fromCurrency (string)  |
| price (decimal)     |       | toCurrency (string)    |
| currency (string)   |       | rate (decimal)         |
| source (string)     |       | source (string)        |
| fetchedAt (datetime)|       | fetchedAt (datetime)   |
+---------------------+       +------------------------+

+---------------------+       +------------------------+
| NetWorthSnapshot    |       | SyncLog                |
|---------------------|       |------------------------|
| id (PK, UUID)       |       | id (PK, UUID)          |
| totalEur (decimal)  |       | exchangeId (FK)        |
| breakdown (jsonb)   |       | status (enum)          |
| snapshotDate (date) |       | startedAt (datetime)   |
| createdAt           |       | completedAt (datetime) |
+---------------------+       | errorMessage (text)    |
                              | assetsSynced (int)     |
                              +------------------------+
```

**Key design notes:**

- `Asset.exchangeId` is nullable -- manual assets (bank accounts, stocks held outside exchanges) have no exchange link. Synced crypto assets link to their ExchangeConnection.
- `AllocationTarget.parentTargetId` creates a three-level hierarchy: macro targets are root nodes, liquidity targets are children of the "Liquidity" macro target, intra-category targets are children of their respective macro category.
- `NetWorthSnapshot.breakdown` is a JSONB column storing the full allocation breakdown at snapshot time. This avoids joins to historical asset states.
- All monetary values use PostgreSQL `DECIMAL(20, 8)` for crypto precision. Never FLOAT.

---

## API Surface (REST)

```
Assets:
  GET    /api/assets              # List all assets
  POST   /api/assets              # Create manual asset
  PUT    /api/assets/:id          # Update asset
  DELETE /api/assets/:id          # Soft-delete asset

Exchanges:
  GET    /api/exchanges           # List CCXT connections
  POST   /api/exchanges           # Add exchange connection
  PUT    /api/exchanges/:id       # Update connection (rotate keys)
  DELETE /api/exchanges/:id       # Remove connection

Sync:
  GET    /api/sync/status         # Last sync status per exchange
  POST   /api/sync/trigger        # Manual trigger for all exchanges
  POST   /api/sync/trigger/:id    # Manual trigger for one exchange

Market Data:
  GET    /api/market-data/:symbol # Current price for symbol

Portfolio:
  GET    /api/portfolio/net-worth         # Current total net worth
  GET    /api/portfolio/allocation        # Allocation breakdown
  GET    /api/portfolio/rebalancing       # Rebalancing suggestions
  GET    /api/portfolio/history           # Historical snapshots
  GET    /api/portfolio/targets           # Current allocation targets
  PUT    /api/portfolio/targets           # Update allocation targets

Dashboard:
  GET    /api/dashboard                  # Aggregated dashboard data
```

---

## Suggested Build Order

Dependencies between components determine what must be built first. Each phase builds on the previous one.

### Phase 1: Foundation (must be first)
**Backend skeleton + database + Docker**

- NestJS project scaffold with module structure
- PostgreSQL + TypeORM configuration with migrations
- Docker compose (backend + DB + basic frontend container)
- Config module (environment variables)
- Common utilities (encryption helper, error filters)

**Rationale:** Everything else depends on the database being available and the project structure being in place. Getting Docker working early prevents "works on my machine" issues.

### Phase 2: Manual Assets (core data model)
**Assets module + basic React frontend**

- Asset entity and CRUD API
- AssetType enum (bank_current, bank_deposit, emergency_fund, stock, etf, etp)
- Basic React app with routing and layout
- Assets page: list, add, edit, delete

**Rationale:** Manual assets are the simplest data flow (no external APIs). They validate the full stack works end-to-end and establish the data model that everything else builds on.

### Phase 3: Market Data Integration
**MarketData module + price display**

- Market data service (Yahoo Finance or equivalent for stocks/ETFs/ETPs)
- Price cache with TTL
- FX rate fetching (USD/EUR, crypto/EUR)
- Update Assets API to include current prices
- Display current values on frontend

**Rationale:** Portfolio valuation depends on prices. This module is needed before any meaningful net worth calculation.

### Phase 4: CCXT Exchange Sync
**Exchanges + Sync modules**

- ExchangeConnection entity and CRUD (with encrypted keys)
- CCXT integration service (fetchBalance)
- Scheduled sync (cron) + manual trigger endpoint
- Sync log tracking
- Map fetched balances to Asset entities
- Frontend: exchange configuration page, sync status

**Rationale:** CCXT sync is the most complex external integration. It depends on Assets existing (to write fetched data) and on MarketData (to price crypto holdings). Building it after both ensures it can be tested with real price data.

### Phase 5: Portfolio Analytics
**Portfolio module + Dashboard aggregation**

- Allocation computation (group assets by category, compute percentages)
- Net worth calculation (sum all assets converted to EUR)
- Allocation targets (CRUD for three-level hierarchy)
- Rebalancing suggestions engine
- Emergency fund status check
- Net worth snapshot creation (daily cron)
- Dashboard API (aggregation endpoint)
- Frontend: full dashboard with charts, history page

**Rationale:** This is the value layer. It depends on all data being available (manual assets + synced crypto + market prices). Building it last means it works with real data from day one.

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Fat Dashboard Controller
**What:** Putting business logic in the dashboard controller/service instead of in domain services.
**Why bad:** The dashboard becomes a god object that knows about CCXT, market data, allocation rules. Impossible to test or reuse.
**Instead:** Dashboard service only composes data from other services. All logic lives in domain modules. Dashboard module has zero entities.

### Anti-Pattern 2: Calling External APIs on Every Request
**What:** Fetching CCXT balances or market prices inline during GET requests.
**Why bad:** External API calls are slow (100ms-2s). A dashboard load would take 5-10 seconds. Rate limits get hit quickly.
**Instead:** Scheduled sync populates the database. GET endpoints only read from the database. Manual sync trigger is a POST that returns immediately and runs async.

### Anti-Pattern 3: Storing API Keys in Plaintext
**What:** Saving exchange API keys directly in the database without encryption.
**Why bad:** Database backups, Docker volumes, and debug logs may expose keys. Even for a personal app, this is negligent.
**Instead:** AES-256 encryption at rest. Key from environment variable. Only decrypt in SyncService at runtime.

### Anti-Pattern 4: Using FLOAT for Monetary Values
**What:** Using JavaScript `number` or PostgreSQL `FLOAT` for asset quantities or prices.
**Why bad:** Floating point arithmetic produces `0.1 + 0.2 = 0.30000000000000004`. Crypto quantities need 8 decimal places. Prices need precision.
**Instead:** PostgreSQL `DECIMAL(20, 8)` columns. Backend uses `string` representation or a decimal library (e.g., `decimal.js`) for calculations. Never use native `number` for money.

### Anti-Pattern 5: Circular Module Dependencies
**What:** Assets module imports Portfolio module, Portfolio module imports Assets module.
**Why bad:** NestJS cannot resolve circular module dependencies without `forwardRef`, which is a code smell indicating bad boundaries.
**Instead:** Portfolio module imports Assets module (reads assets). Assets module never imports Portfolio module. Data flows one direction.

### Anti-Pattern 6: Frontend Polling
**What:** React components polling backend endpoints every few seconds.
**Why bad:** Unnecessary load for a single-user personal finance app. Data changes on 15-minute sync intervals, not real-time.
**Instead:** Fetch on page load. Manual refresh button after triggering sync. Consider Server-Sent Events only if the user explicitly requests live sync feedback.

---

## Scaling Considerations

This is a single-user personal finance app. Scaling beyond one user is explicitly out of scope. However, there are practical performance considerations:

| Concern | Single User Impact | Mitigation |
|---------|-------------------|------------|
| CCXT API rate limits | 15-min sync interval for ~5 exchanges = ~20 API calls per sync, well within limits | Exponential backoff on 429 errors |
| Market data API limits | ~50 stock/ETF symbols with 15-min cache = ~200 calls/day | Batch requests where API supports it |
| Dashboard load time | Single aggregation query + FX lookups, ~100ms expected | Parallel async queries in DashboardService |
| Net worth snapshots | 365 rows/year, negligible | No optimization needed |
| Database size | <1000 assets, <10K snapshots over 10 years | PostgreSQL handles this trivially |

No caching layer (Redis), no message queue, no read replicas needed. The entire system fits comfortably on a single Docker host.

---

## Sources

- NestJS Modules documentation (Context7, /nestjs/docs.nestjs.com) -- HIGH confidence
- NestJS TypeORM integration (Context7, /nestjs/typeorm) -- HIGH confidence
- CCXT balance fetching and error handling (Context7, /ccxt/ccxt) -- HIGH confidence
- NestJS Task Scheduling with @Cron (Context7, /nestjs/docs.nestjs.com) -- HIGH confidence
- react-chartjs-2 for dashboard charts (Context7, /reactchartjs/react-chartjs-2) -- HIGH confidence
- Project requirements from .planning/PROJECT.md -- HIGH confidence
