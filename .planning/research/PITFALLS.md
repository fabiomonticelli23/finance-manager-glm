# Domain Pitfalls

**Domain:** Personal finance portfolio management (CCXT crypto, multi-currency, rebalancing)
**Researched:** 2026-04-20
**Confidence:** HIGH (CCXT, TypeORM, NestJS Schedule sourced from Context7; yahoo-finance2 error handling from Context7; FX/rebalancing patterns from domain expertise)

---

## Critical Pitfalls

Mistakes that cause rewrites, data loss, or fundamental architecture changes.

### CP-1: Storing Monetary Values as JavaScript Floating-Point Numbers

**What goes wrong:** JavaScript `number` is IEEE 754 double-precision. `0.1 + 0.2 !== 0.3`. Over many additions and currency conversions, portfolio totals drift. A 100K EUR portfolio silently reports 99999.99999998 or 100000.00000001. Compounding across historical snapshots, the drift becomes visible and irreparable.

**Why it happens:** TypeORM defaults `@Column()` to integer for number properties. Developers set `type: "float"` or `type: "double"` thinking it is precise enough for money. It is not.

**Consequences:** Silent accumulation errors in net worth calculations. Rebalancing suggestions based on wrong percentages. Historical trend lines with visible jitter. Full migration required to fix (every monetary column, every calculation).

**Prevention:**
- Use PostgreSQL `NUMERIC(precision, scale)` or `DECIMAL(precision, scale)` for all monetary columns
- TypeORM definition: `@Column({ type: 'decimal', precision: 20, scale: 8 })` -- 20 total digits, 8 after decimal (covers crypto Satoshi-precision and fiat)
- In TypeORM, the underlying `pg` driver returns `DECIMAL` columns as strings by default. Keep them as strings through the backend, parse to `BigDecimal` or use a library like `decimal.js` for arithmetic
- Never use JavaScript `+`, `-`, `*`, `/` for monetary math. Use `decimal.js` or equivalent
- Define a shared `Money` value object: `{ amount: string, currency: string }` and enforce it through the codebase

**Detection:** Unit tests that assert `addMoney(0.1, 0.2) === 0.3` will fail with native numbers. If no such tests exist, the pitfall has likely already landed.

**Phase:** Phase 1 (data model and database schema) -- this is foundational, impossible to fix cleanly later.

**Sources:** TypeORM Context7 docs confirm `decimal` type support in PostgreSQL and `bigNumberStrings` behavior; JavaScript IEEE 754 is well-documented.

---

### CP-2: Not Distinguishing CCXT NetworkError from ExchangeError

**What goes wrong:** All CCXT errors are caught with a single `catch (e)` handler. Network blips (timeout, DNS failure) are treated the same as invalid API keys or insufficient permissions. The system either retries authentication errors indefinitely (hammering the exchange) or silently drops transient network failures (stale portfolio data).

**Why it happens:** CCXT's error hierarchy is not widely known. `BaseError` splits into `NetworkError` (retryable) and `ExchangeError` (non-retryable). Developers who do not read the CCXT docs treat all errors as equivalent.

**Consequences:** Exchange API bans from retry storms on auth errors. Stale balances shown to user without warning. Alert fatigue from logging every network hiccup as a critical error.

**Prevention:**
```
NetworkError (retry with exponential backoff)
  ├── RequestTimeout        --> retry
  ├── ExchangeNotAvailable   --> retry
  ├── RateLimitExceeded      --> back off longer, then retry
  └── DDoSProtection         --> back off significantly, then retry

ExchangeError (do NOT retry, surface to user)
  ├── AuthenticationError    --> invalid API key, alert user
  ├── InsufficientFunds      --> data inconsistency, log warning
  ├── InvalidOrder           --> not applicable here (no trading)
  └── NotSupported           --> exchange does not support this endpoint
```

- Implement a CCXT error classifier utility that maps each error subclass to a recovery strategy
- Use exponential backoff with jitter for NetworkError retries (max 3 attempts)
- Surface AuthenticationError in the UI immediately ("Exchange connection broken -- check API keys")
- Log RateLimitExceeded events to detect polling-interval misconfiguration

**Detection:** If the sync service has a single `catch (error)` block with no instanceof checks, this pitfall is present.

**Phase:** Phase 2 (CCXT integration) -- must be correct from the first exchange connection.

**Sources:** CCXT Context7 docs (error hierarchy section), HIGH confidence.

---

### CP-3: Storing API Keys Unencrypted at Rest

**What goes wrong:** CCXT exchange API keys (and secrets) are stored as plaintext in PostgreSQL. Anyone with database access (backup leak, Docker volume exposure, localhost port open) has full exchange API keys. Even read-only keys leak balance information and, on some exchanges, can be escalated.

**Why it happens:** NestJS + TypeORM tutorials never cover encryption. Developers store secrets in a `.env` file and think that is sufficient. But `.env` is for config, not per-row user data. Exchange API keys are per-exchange-connection data that must be stored in the database, encrypted.

**Consequences:** Full exposure of financial API keys on any database leak. Cannot rotate keys from the app (keys are not recoverable once encrypted if the approach is wrong). Regulatory risk depending on jurisdiction.

**Prevention:**
- Encrypt API keys at rest using AES-256-GCM with a master encryption key stored in environment variables (not in the database)
- Use NestJS's `ConfigService` to inject the encryption key at startup
- Store `iv` + `authTag` + `ciphertext` in separate columns or a single JSONB column alongside each exchange connection record
- Provide a key-rotation mechanism: decrypt with old key, re-encrypt with new key
- The master key should be in Docker secrets or a `.env` file that is git-ignored and never committed
- Consider using `crypto.createCipheriv` / `crypto.createDecipheriv` from Node.js built-in `crypto` module -- no external dependency needed

**Detection:** If the `ExchangeCredential` entity has `@Column() apiKey: string` without an encryption transform, this pitfall is present.

**Phase:** Phase 2 (CCXT integration) -- security must be baked in from the start, not retrofitted.

**Sources:** Node.js crypto module documentation; domain expertise.

---

### CP-4: Single Currency Assumption in Data Model

**What goes wrong:** The database schema stores `amount` without a `currency` column, or hardcodes EUR everywhere. When the user adds a USD-denominated stock or a BTC holding, there is no way to represent the original currency. Every value is immediately converted to EUR on write, losing the source currency. When exchange rates change, historical snapshots cannot be re-evaluated.

**Why it happens:** EUR is the "base currency" in the project spec. Developers interpret this as "store everything in EUR." Base currency means "display in EUR," not "discard source currency."

**Consequences:** Cannot track FX gains vs asset gains separately. Historical net worth is wrong if FX rates were stale at snapshot time and cannot be corrected. Adding a new currency requires a full schema migration and data backfill.

**Prevention:**
- Every monetary value in the database stores both `amount` (decimal) and `currency` (string, ISO 4217 or crypto symbol)
- Conversion to EUR happens at read time for display, or at snapshot time alongside the original value
- Historical snapshots store both the original-currency amount and the EUR-converted amount, plus the exchange rate used
- Define a `Currency` enum/type covering all supported currencies and crypto symbols
- FX rates are cached with timestamps; never use stale rates for conversion without marking staleness

**Detection:** If any entity has `amount` without an adjacent `currency` column, this pitfall is present.

**Phase:** Phase 1 (data model) -- schema migration after launch is painful.

**Sources:** Domain expertise; standard multi-currency financial application patterns.

---

### CP-5: Rebalancing Math That Ignores Drift Tolerance

**What goes wrong:** Rebalancing suggestions show "move 0.01% from Stocks to Crypto" because the current allocation is 34.99% and the target is 35.00%. The user is bombarded with trivial rebalancing suggestions that are noise, not signal. Worse: rebalancing suggestions change every minute because market prices fluctuate, making the feature unusable.

**Why it happens:** Developers implement exact percentage matching without considering transaction costs, minimum trade sizes, or meaningful deviation thresholds.

**Consequences:** Users ignore the rebalancing feature entirely (alert fatigue). Trust erosion -- if suggestions are trivial now, maybe the important ones are wrong too.

**Prevention:**
- Define drift tolerance bands per allocation level: macro (e.g., 5%), liquidity (e.g., 3%), intra-category (e.g., 5%)
- Only surface rebalancing suggestions when actual allocation drifts beyond the tolerance band
- Weight suggestions by impact: prioritize moves that bring multiple allocations back in range
- Consider a "minimum rebalance amount" threshold (e.g., do not suggest moves under 100 EUR)
- Cache rebalancing calculations; do not recalculate on every dashboard load if prices have not changed significantly

**Detection:** If the rebalancing endpoint returns suggestions for sub-1% deviations, this pitfall is present.

**Phase:** Phase 3 (rebalancing logic) -- the core feature must be useful, not technically correct but practically useless.

**Sources:** Domain expertise; portfolio management literature on "rebalancing bands" vs "calendar rebalancing."

---

## Technical Debt Patterns

Patterns that accumulate if not addressed early.

### TDP-1: CCXT Exchange Instance Not Reused

**What goes wrong:** A new CCXT exchange instance is created on every sync cycle. Each instantiation triggers `loadMarkets()` (or the first API call does), adding 200-500ms of latency per exchange. With 5 exchanges and a 1-minute sync interval, that is 1-2.5 seconds of pure overhead per cycle, plus unnecessary rate-limit consumption.

**Prevention:**
- Create a singleton `ExchangeInstanceManager` that caches CCXT exchange instances by exchange ID + credentials hash
- Call `exchange.loadMarkets()` explicitly after instantiation, not lazily
- Re-instantiate only when credentials change or after a configurable TTL (e.g., 24 hours) to pick up new market listings
- NestJS singleton services are the natural home for this: `@Injectable({ scope: Scope.DEFAULT })`

**Detection:** If `new ccxt.binance({...})` appears inside a method that is called repeatedly, this debt is accumulating.

**Phase:** Phase 2 (CCXT integration).

**Sources:** CCXT Context7 docs on loadMarkets() behavior; HIGH confidence.

---

### TDP-2: Polling All Exchanges on a Single Fixed Schedule

**What goes wrong:** All exchanges are polled every N minutes on the same cron schedule. Some exchanges (e.g., Binance) have generous rate limits and can be polled frequently. Others (e.g., CoinGecko for prices, or smaller exchanges) have strict limits. A uniform schedule either over-polls slow endpoints (rate limit bans) or under-polls fast endpoints (stale data).

**Prevention:**
- Make the polling interval configurable per exchange connection, stored in the database
- Default: 5 minutes for exchange balances, 15 minutes for market prices
- Use NestJS `SchedulerRegistry` to dynamically add/remove cron jobs when exchange connections are added/removed
- Stagger the cron schedules so not all exchanges sync at the same second
- Implement a "last synced" timestamp per exchange; skip if last sync succeeded recently

**Detection:** If there is a single `@Cron('*/5 * * * *')` that iterates all exchanges, this pattern is present.

**Phase:** Phase 2 (CCXT integration, scheduling).

**Sources:** NestJS Schedule Context7 docs; CCXT rate limit docs; HIGH confidence.

---

### TDP-3: No Idempotency on Historical Snapshot Creation

**What goes wrong:** The scheduled job that creates net-worth historical snapshots runs, the server restarts, and the job re-runs, creating duplicate snapshots for the same timestamp. Over time, historical trend charts show duplicate data points or require deduplication logic that was never built.

**Prevention:**
- Use a composite unique constraint: `(snapshot_date, snapshot_type)` or `(timestamp_truncated_to_hour, source)`
- Use `INSERT ... ON CONFLICT DO NOTHING` (PostgreSQL upsert) for snapshot creation
- Record the job execution in a `sync_log` table: `{ exchange_id, started_at, completed_at, status }`
- Before creating a snapshot, check if one already exists for the current period

**Detection:** If the snapshot table has no unique constraints on date/time columns, this debt is accumulating.

**Phase:** Phase 1 (data model) and Phase 2 (sync service).

**Sources:** PostgreSQL documentation; domain expertise.

---

## Integration Gotchas

### IG-1: CCXT loadMarkets() Double-Request on First API Call

**What goes wrong:** If `loadMarkets()` is not called explicitly, the first API call (e.g., `fetchBalance()`) triggers an implicit `loadMarkets()` call, doubling the first request time. Users see slow first loads and think the integration is broken.

**Prevention:**
- Always call `await exchange.loadMarkets()` explicitly after creating an exchange instance
- Do this during the exchange connection validation step (when the user adds API keys)
- Log the loadMarkets() timing to detect exchanges with unusually large market lists

**Detection:** First API call latency is 2x-3x subsequent calls.

**Phase:** Phase 2 (CCXT integration).

**Sources:** CCXT Context7 docs, HIGH confidence.

---

### IG-2: CCXT Balance Structure Inconsistency Across Exchanges

**What goes wrong:** CCXT returns balances as `{ BTC: { free: 0.5, used: 0.1, total: 0.6 } }`. However, not all exchanges report all three fields consistently. Some exchanges report `used` as 0 even when there are open orders. Others report `total` but not `free` and `used` separately. The code assumes all three are always present and non-zero.

**Prevention:**
- Always use `total` as the authoritative balance figure for net worth calculations
- Compute `used = total - free` if `used` is missing or zero but `total` and `free` differ
- Log warnings when balance structure deviates from expected format
- Write per-exchange integration tests with mocked balance responses that cover edge cases (missing fields, zero values)

**Detection:** Spot-check `free + used !== total` for various exchanges in test data.

**Phase:** Phase 2 (CCXT integration).

**Sources:** CCXT Context7 docs on balance structure; HIGH confidence.

---

### IG-3: Yahoo Finance Rate Limiting and Validation Errors

**What goes wrong:** yahoo-finance2 has a built-in concurrency limit of 4 simultaneous requests. If the app tries to fetch prices for 200 symbols simultaneously, requests queue and some time out. Additionally, Yahoo Finance returns data that sometimes fails yahoo-finance2's own schema validation, throwing `FailedYahooValidationError`. The app crashes or returns empty price data.

**Prevention:**
- Let yahoo-finance2's built-in concurrency limiter do its job -- do not wrap calls in `Promise.all()` with 200 symbols
- Always wrap `yahooFinance.quote()` calls in try/catch and handle `HTTPError` (log and skip) separately from `FailedYahooValidationError` (use `error.result` for partial data)
- Batch symbol requests using yahoo-finance2's `quoteCombine()` utility for high-volume fetching
- Cache prices with a TTL (e.g., 15 minutes for stocks/ETFs) to avoid redundant API calls
- Implement a fallback: if Yahoo Finance is down, serve cached prices with a "stale" indicator

**Detection:** If `yahooFinance.quote()` calls are not wrapped in try/catch, or if `Promise.all()` is called with more than 4 symbols, this gotcha will hit.

**Phase:** Phase 2 (market data integration).

**Sources:** yahoo-finance2 Context7 docs; HIGH confidence.

---

### IG-4: CCXT Rate Limit Consumption by fetchTickers Without Symbols

**What goes wrong:** Calling `exchange.fetchTickers()` (no arguments) fetches all tickers for all markets on the exchange. On Binance, that is 2000+ tickers in a single request. On smaller exchanges, the request itself is slow, and the response is massive. Doing this on every sync cycle burns through rate limits for data that is mostly irrelevant.

**Prevention:**
- Always call `exchange.fetchTickers([symbols])` with only the symbols the user actually holds
- Maintain a "watched symbols" list derived from the user's portfolio
- If the user has no positions on a given exchange, skip the fetchTickers call entirely (use fetchBalance to check for non-zero balances)

**Detection:** Network logs show multi-MB responses from fetchTickers calls.

**Phase:** Phase 2 (CCXT integration).

**Sources:** CCXT Context7 docs; HIGH confidence.

---

### IG-5: FX Conversion Staleness

**What goes wrong:** The app fetches a EUR/USD rate once at startup or infrequently, then uses it for all conversions. Crypto prices move in minutes; FX rates for major pairs are relatively stable but still shift intra-day. A stale FX rate means net worth displayed in EUR is wrong by potentially hundreds of euros for large portfolios.

**Prevention:**
- Fetch FX rates on a schedule separate from asset prices (e.g., every 30 minutes for major fiat pairs)
- Store FX rates with timestamps; mark any rate older than 1 hour as "stale" in the UI
- Use the same FX rate for all conversions within a single snapshot to ensure consistency
- Never hardcode FX rates or assume 1:1 parity

**Detection:** If FX rates table has entries older than 24 hours during active use.

**Phase:** Phase 2 (market data integration).

**Sources:** Domain expertise.

---

## Performance Traps

### PT-1: N+1 Queries in Portfolio Aggregation

**What goes wrong:** The dashboard loads by querying all assets, then for each asset makes a separate query to get the latest price, then another query for the historical snapshot. For 50 assets, that is 100+ database queries per dashboard load. Response time: 2-5 seconds on local, worse over network.

**Prevention:**
- Use a single aggregation query with JOINs: `SELECT a.*, p.latest_price, p.price_currency FROM assets a LEFT JOIN latest_prices p ON ...`
- Create a materialized view or denormalized `portfolio_summary` table that is updated on sync, not on read
- Batch all price lookups into a single query using `WHERE symbol IN (...)`
- Use NestJS caching (`@nestjs/cache-manager`) for the dashboard summary with a short TTL (matches sync interval)

**Detection:** Enable TypeORM query logging in development. If dashboard load produces 20+ queries, this trap is active.

**Phase:** Phase 1 (data model design) and Phase 3 (dashboard optimization).

**Sources:** TypeORM query patterns; domain expertise.

---

### PT-2: Dashboard Recalculating Everything on Every Page Load

**What goes wrong:** The net worth, allocation percentages, and drift calculations are all computed from raw asset data on every GET /dashboard request. With historical data, this involves scanning thousands of rows. The dashboard takes 3-10 seconds to load, and the user sees a spinner every time they switch tabs.

**Prevention:**
- Pre-compute and cache the dashboard summary after each sync cycle
- Store computed values in a `dashboard_cache` table or Redis: `{ total_net_worth, allocations_json, drift_json, computed_at }`
- Dashboard endpoint reads from cache; background sync job updates the cache
- If no sync has happened since last dashboard load, return cached data (sub-100ms response)
- Invalidate cache only when a sync completes or a manual asset is edited

**Detection:** Dashboard response time exceeds 500ms consistently.

**Phase:** Phase 3 (dashboard and analytics).

**Sources:** Domain expertise; standard caching patterns.

---

### PT-3: Unbounded Historical Data Queries

**What goes wrong:** The historical net worth chart queries all snapshots from the beginning of time. After a year of 5-minute snapshots, that is ~105K rows. The chart endpoint becomes slow, and the frontend tries to render 105K data points on a line chart, crashing the browser tab.

**Prevention:**
- Downsample historical data for chart rendering: hourly for the last 7 days, daily for the last year, weekly for older data
- Use PostgreSQL `date_trunc()` for server-side aggregation: `SELECT date_trunc('day', timestamp) AS day, AVG(net_worth_eur) FROM snapshots GROUP BY day`
- Implement pagination or date-range filtering on the historical data endpoint
- Consider a separate aggregation table for pre-computed daily/weekly/monthly averages

**Detection:** Historical chart endpoint response size exceeds 1MB or response time exceeds 2 seconds.

**Phase:** Phase 3 (historical tracking and charts).

**Sources:** PostgreSQL documentation; domain expertise.

---

## Security Mistakes

### SM-1: Exposing Exchange API Keys in Error Messages or Logs

**What goes wrong:** When a CCXT call fails, the error object or the logged request URL contains the API key as a query parameter or header. Logs are written to disk (or stdout in Docker) without sanitization. Anyone with access to Docker logs has the exchange API keys.

**Prevention:**
- Never log the full CCXT error object; extract only `message` and `name`
- Configure CCXT with `exchange.verbose = false` in production (verbose mode logs full request/response)
- Use a custom NestJS logger interceptor that redacts patterns matching common API key formats
- In Docker, configure log rotation and do not persist logs to volumes
- If API keys must appear in URLs (some exchanges do this), ensure they are stripped before logging

**Detection:** Search codebase for `console.log(error)` or `this.logger.log(error)` inside CCXT call catch blocks.

**Phase:** Phase 2 (CCXT integration).

**Sources:** CCXT docs on verbose mode; domain expertise.

---

### SM-2: No CORS or Network Restrictions on Solo-User API

**What goes wrong:** The NestJS backend is exposed on 0.0.0.0:3000 with no CORS restrictions because "it is a single-user app." Anyone on the same network (or the internet if the machine is exposed) can read all financial data, and worse, add/modify/delete exchange connections and manual assets.

**Prevention:**
- Even without authentication, restrict CORS to the frontend origin only: `app.enableCors({ origin: 'http://localhost:5173' })` (or the Docker network frontend URL)
- Bind the backend to the Docker internal network only; do not expose the backend port to the host machine
- In `docker-compose.yml`, use internal networks so the backend is only reachable from the frontend container
- Consider a simple bearer token in `.env` for API access -- not full auth, but prevents accidental exposure
- The frontend should be the only thing that can reach the backend

**Detection:** Backend is accessible from the host machine via `curl http://localhost:3000/api/assets`.

**Phase:** Phase 1 (project scaffolding, Docker setup).

**Sources:** NestJS CORS documentation; Docker networking documentation.

---

### SM-3: Storing CCXT Secrets in Environment Variables for Per-Exchange Config

**What goes wrong:** Since there can be multiple exchange connections (each with its own API key/secret), developers try to store all of them in `.env`: `BINANCE_API_KEY=...`, `KRAKEN_API_KEY=...`, etc. This breaks when the user adds a new exchange -- they must edit `.env` and restart the container. It also means all keys are in one file, all unencrypted.

**Prevention:**
- Store exchange credentials in the database (encrypted, per CP-3)
- Use `.env` only for the database connection string and the encryption master key
- Provide a UI or API endpoint to add/remove exchange connections without restarting the container
- The CCXT exchange instance manager reads credentials from the database at runtime, not from env vars

**Detection:** `.env` file contains `*_API_KEY` or `*_SECRET` entries for specific exchanges.

**Phase:** Phase 2 (CCXT integration).

**Sources:** Domain expertise.

---

## UX Pitfalls

### UX-1: Showing Raw Crypto Amounts Without Human-Readable Formatting

**What goes wrong:** The dashboard shows `0.00042310 BTC` or `1.50000000 ETH`. The user cannot quickly assess whether 0.00042310 BTC is $40 or $400. Worse, EUR amounts are shown as `12345.67000000` instead of `12,345.67 EUR`.

**Prevention:**
- Format all monetary values with `Intl.NumberFormat` using appropriate currency locale
- For crypto: show both the crypto amount (4-8 decimal places) and the EUR equivalent prominently
- Always display the EUR value more prominently than the crypto amount for the base-currency view
- Use consistent formatting: EUR always 2 decimals, crypto varies by unit value
- Consider: `12,345.67 EUR (0.1234 BTC)` rather than just one or the other

**Detection:** Any dashboard element shows more than 2 decimal places for EUR values, or shows crypto without EUR equivalent.

**Phase:** Phase 3 (dashboard UI).

**Sources:** Domain expertise; `Intl.NumberFormat` MDN documentation.

---

### UX-2: No Loading/Error State for Stale Exchange Data

**What goes wrong:** Exchange sync fails silently. The dashboard shows the last-known balances with no indication that data is 3 hours stale. The user makes rebalancing decisions based on outdated information. Or worse, the dashboard shows a blank screen with no error message because the API returned a 500.

**Prevention:**
- Every dashboard section shows a "Last synced: [relative time]" indicator
- If data is older than 2x the expected sync interval, show a visual warning (yellow banner, stale badge)
- If a sync fails, show the error in the UI: "Binance sync failed: Invalid API key. Last successful sync: 2h ago."
- Never show an empty dashboard -- always show cached data with staleness indicator
- Implement skeleton loading states for initial load (before any data exists)

**Detection:** Dashboard sections have no timestamp metadata, or sync failures produce no visual feedback.

**Phase:** Phase 3 (dashboard UI).

**Sources:** Domain expertise.

---

### UX-3: Emergency Fund Status Not Visually Prominent

**What goes wrong:** The emergency fund is listed as just another line item in the Liquidity section. The user does not notice that their emergency fund has dropped below the target. The requirement says "highlighted when underfunded or overfunded" but this is implemented as subtle text color rather than a clear visual indicator.

**Prevention:**
- Emergency fund status deserves its own dashboard card, not a row in a table
- Use distinct visual states: green (funded), yellow (within 10% of target), red (underfunded by more than 10%)
- Show both absolute amount and percentage of target
- Include the shortfall/surplus amount: "Emergency fund: 8,500 EUR (85% of 10,000 EUR target). Shortfall: 1,500 EUR."
- This is a core value proposition of the app -- treat it with corresponding visual weight

**Detection:** Emergency fund status is not immediately visible when opening the dashboard without scrolling.

**Phase:** Phase 3 (dashboard UI, rebalancing features).

**Sources:** PROJECT.md requirements; domain expertise.

---

## Pitfall-to-Phase Mapping

### Phase 1: Foundation (Data Model, Docker, Scaffolding)

| Pitfall ID | Pitfall | Why Here |
|------------|---------|----------|
| CP-1 | Monetary values as float | Schema decision; impossible to fix cleanly later |
| CP-4 | Single currency assumption | Schema must include currency columns from day one |
| TDP-3 | No snapshot idempotency | Unique constraints defined in initial migration |
| PT-1 | N+1 queries | Data model design determines query patterns |
| SM-2 | No CORS/network restrictions | Docker network topology set at scaffolding |

### Phase 2: CCXT Integration and Market Data

| Pitfall ID | Pitfall | Why Here |
|------------|---------|----------|
| CP-2 | NetworkError vs ExchangeError | Error handling must be correct from first exchange connection |
| CP-3 | Unencrypted API keys | Security must be baked in, not retrofitted |
| TDP-1 | CCXT instance not reused | Exchange instance manager is a core integration component |
| TDP-2 | Uniform polling schedule | Scheduling logic is part of sync service design |
| IG-1 | loadMarkets() double request | Must be handled in exchange initialization |
| IG-2 | Balance structure inconsistency | Must be handled in balance normalization layer |
| IG-3 | Yahoo Finance rate limiting | Market data integration error handling |
| IG-4 | fetchTickers without symbols | Rate limit management in sync service |
| IG-5 | FX conversion staleness | FX rate fetching is part of market data pipeline |
| SM-1 | API keys in logs | Logging must be sanitized from the start |
| SM-3 | Exchange creds in .env | Credential storage architecture decision |

### Phase 3: Dashboard, Rebalancing, Analytics

| Pitfall ID | Pitfall | Why Here |
|------------|---------|----------|
| CP-5 | Rebalancing without drift tolerance | Core rebalancing logic must be useful from the start |
| PT-2 | Dashboard recalculating on every load | Caching strategy implemented with dashboard |
| PT-3 | Unbounded historical queries | Chart rendering and query optimization |
| UX-1 | Raw number formatting | Frontend presentation layer |
| UX-2 | No stale data indicator | Dashboard state management |
| UX-3 | Emergency fund not prominent | Dashboard layout and visual design |

### Cross-Phase Warnings

| Topic | Warning | Phase Affected |
|-------|---------|----------------|
| TypeORM decimal handling | PostgreSQL returns DECIMAL as strings; do not `.parseFloat()` them | 1, 2, 3 |
| CCXT Pro vs Free | Free CCXT is REST-only; WebSocket (watchBalance) requires CCXT Pro (paid) | 2 |
| Docker networking | Backend should not be exposed to host; use Docker internal networks | 1 |
| Testing with real exchanges | Never test against real exchange APIs with real keys; use CCXT's sandbox mode or mocked responses | 2 |
| yahoo-finance2 stability | Yahoo Finance is an unofficial API; it can break without warning. Have fallback caching | 2 |

## Sources

- CCXT documentation (Context7, library ID /ccxt/ccxt): Error hierarchy, rate limiting, balance structure, loadMarkets, precision -- HIGH confidence
- TypeORM documentation (Context7, library ID /typeorm/typeorm): Decimal/numeric column types, bigNumberStrings, PostgreSQL driver -- HIGH confidence
- NestJS Schedule documentation (Context7, library ID /nestjs/docs.nestjs.com): Cron decorators, SchedulerRegistry, dynamic jobs -- HIGH confidence
- yahoo-finance2 documentation (Context7, library ID /gadicc/yahoo-finance2): Error types, concurrency limits, validation errors -- HIGH confidence
- JavaScript IEEE 754 floating-point: Well-documented, universal knowledge -- HIGH confidence
- Portfolio rebalancing theory: Domain expertise, standard financial literature -- MEDIUM confidence (recommend validating drift tolerance values with user)
- Multi-currency FX patterns: Domain expertise, standard financial application patterns -- MEDIUM confidence
