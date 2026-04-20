# Feature Landscape

**Domain:** Personal finance portfolio management (single-user)
**Researched:** 2026-04-20

## Table Stakes

Features users expect from a portfolio tracker. Missing these means the product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Total net worth dashboard | Core value proposition: single number across all assets | Low | Sum of all asset values converted to EUR. Straightforward arithmetic once data sources are connected. |
| Multi-currency conversion to base currency (EUR) | Eurozone user with USD stocks and crypto; conversion is non-negotiable | Medium | yahoo-finance2 supports forex pairs (e.g., `EURUSD=X`). Cache rates; refresh with price polling cycle. |
| Crypto exchange balance import via CCXT | Primary data source for crypto holdings; manual entry defeats the purpose | Medium | CCXT `fetchBalance()` returns free/used/total per currency. Enable `enableRateLimit: true`. Handle exchange-specific quirks per CCXT docs. |
| Manual asset entry (bank accounts, stocks, ETFs, ETPs) | Not everything has an API; bank accounts and some holdings are manual by nature | Low | CRUD for asset records with fields: name, type, quantity, cost basis, currency, institution. Standard form + table UI. |
| Auto-fetched market prices for tradable assets | Stale prices make the dashboard useless; yahoo-finance2 provides this | Medium | yahoo-finance2 `quote()` returns `regularMarketPrice` and `currency`. Map ticker symbols to assets. Handle market closed vs. after-hours. |
| Portfolio allocation pie chart | Every portfolio tracker has this; users need visual breakdown | Low | Recharts `PieChart` with `ResponsiveContainer`. Group assets by macro category (Crypto, Stocks, Liquidity, Trading Bots). |
| Historical net worth trend line | Users need to see progress over time; flat numbers lack context | Medium | Requires daily snapshot persistence in PostgreSQL. Recharts `LineChart` with time axis. Snapshot job runs on schedule (NestJS `@Cron`). |
| Auto-polling data sync on schedule | Manual refresh-only is unacceptable for a monitoring tool | Low | NestJS `@Cron` decorator for periodic sync. CCXT balances + yahoo-finance2 prices in one scheduled job. |
| Manual refresh on demand | Users want control; cannot wait for next poll cycle | Low | Button in UI triggers backend endpoint. Same sync logic as scheduled job. |
| Asset categorization (macro categories) | Without categories, allocation analysis is impossible | Low | Four macro categories: Crypto, Stocks, Liquidity, Trading Bots. Each asset tagged at creation. Simple enum field. |
| Responsive web layout | Must work on desktop and tablet; web-first does not mean desktop-only | Low | CSS grid/flexbox with breakpoints. Recharts `ResponsiveContainer` handles chart scaling. |

## Differentiators

Features that set this product apart from generic trackers. Not expected by default, but highly valued.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Multi-level rebalancing suggestions | Most trackers show allocation; few tell you what to do about it. Three levels: macro, liquidity, intra-category. | High | Requires target allocation config per level. Drift calculation: `(current% - target%)`. Display as ordered action list. Macro level first (e.g., "move 5% from Crypto to Liquidity"). |
| Emergency fund as protected allocation | Treats emergency fund as a first-class constraint, not just another bank account. Alerts when underfunded. | Medium | Target value or percentage config. Dashboard indicator (green/yellow/red) based on current vs. target. Protected status prevents rebalancer from suggesting moves out of it. |
| Trading Bot tagged accounts | Separates active trading from passive holdings; critical for accurate risk/rebalancing analysis | Medium | CCXT connections tagged with `isTradingBot: true`. Tag changes macro category from Crypto to Trading Bots. Same `fetchBalance()` call, different grouping. |
| Intra-category rebalancing (e.g., BTC vs ETH vs altcoins) | Granular control within a category; most tools stop at macro allocation | High | Requires per-asset target weights within each category. Drift calculation at category level. Actionable: "sell X% of DOGE, buy Y% more BTC." |
| Allocation drift indicators | Visual cue when portfolio has drifted from targets without requiring drill-down | Medium | Color-coded badges on dashboard cards. Threshold-based (e.g., >5% drift = yellow, >10% = red). Computed from current vs. target allocations. |
| Liquidity sub-allocation | Breaks down cash into current accounts, deposit accounts, emergency fund with target ratios | Medium | Separate target config for liquidity category. Useful for cash management discipline. Nested pie chart or bar chart within Liquidity section. |
| Per-asset cost basis tracking | Enables gain/loss calculation per holding; most basic trackers omit this | Low | Store purchase price alongside quantity. Compute unrealized P&L: `(currentPrice - costBasis) * quantity`. Display as green/red in asset list. |

## Anti-Features

Features to explicitly NOT build. These are tempting but wrong for this project's scope.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| Trade execution / order placement | Enormous security liability; CCXT supports it but executing trades requires storing API keys with withdrawal permissions. Single-user tool should not handle this risk. | Rebalancing suggestions only. User executes trades manually on exchange. |
| Multi-user authentication system | Solo user tool. Auth adds complexity (sessions, JWT, password reset, rate limiting) with zero benefit. | Single-user with no login. Consider basic HTTP auth or IP whitelist for deployment if network-exposed. |
| Push notifications / email alerts | Requirement explicitly excludes this. Dashboard indicators suffice for a tool the user checks actively. | Dashboard-only visual indicators (color coding, badges). User sees status when they open the app. |
| Mobile native app | Requirement explicitly web-first. Native app doubles development effort for a single user. | Responsive web design that works well on mobile browsers. PWA installability if desired later. |
| Social / community features | No sharing, following, leaderboards. This is a private financial tool, not a social platform. | No user-facing social features. Data stays local and private. |
| Tax reporting / capital gains export | Complex jurisdiction-specific calculations. Belgium has specific crypto tax rules that change. Out of scope for a portfolio tracker. | Provide raw P&L data per asset. User handles tax calculations or exports to a tax tool. |
| Real-time streaming prices | WebSocket feeds for every asset adds infrastructure complexity (connection management, reconnection, backpressure). Portfolio snapshots are sufficient. | Polling-based price refresh on schedule (every 15-60 minutes). Manual refresh for on-demand updates. |
| News feed / sentiment analysis | Scope creep into financial media. Adds no value to net worth tracking. | No news integration. Pure data and analytics. |

## Feature Dependencies

```
CCXT Integration --> Crypto Balance Import --> Total Net Worth Calculation
                                              |
Manual Asset Entry -------------------------->|
                                              |
Auto-Fetch Market Prices ------------------->|
                                              v
                                    Total Net Worth Dashboard
                                              |
                                              +--> Historical Snapshots --> Trend Charts
                                              |
Multi-Currency Conversion --------------------+--> Allocation Pie Chart
                                              |
Asset Categorization ------------------------->|
                                              |
Target Allocation Config --------------------->+--> Rebalancing Suggestions
                                              |         |
Trading Bot Tagging --------------------------+         +--> Macro Level
                                                        |
                                                        +--> Liquidity Level
                                                        |
Emergency Fund Config --> Emergency Fund Status -------->+--> Intra-Category Level
```

Dependency rules for implementation ordering:

1. **Asset data layer first** -- CCXT integration, manual asset CRUD, price fetching. Without data, nothing else works.
2. **Currency conversion** -- Must be in place before any aggregation. EUR conversion is a prerequisite for net worth calculation.
3. **Aggregation and display** -- Net worth calculation, allocation grouping, charts. Depends on data layer being complete.
4. **Historical tracking** -- Snapshot mechanism. Depends on aggregation being correct (bad data = bad history).
5. **Rebalancing engine** -- Target allocation config + drift math. Depends on accurate allocation calculations.
6. **Multi-level rebalancing** -- Nested target configs. Depends on basic rebalancing working.
7. **Drift indicators** -- Visual layer on top of rebalancing calculations. Last to build.

## MVP Recommendation

**Phase 1 MVP** -- Get data flowing and visible:

1. Manual asset entry (bank accounts, stocks, ETFs, ETPs) -- the foundation
2. CCXT crypto exchange integration -- primary automated data source
3. Auto-fetch market prices via yahoo-finance2 -- keeps data current
4. Multi-currency conversion to EUR -- non-negotiable for aggregation
5. Total net worth dashboard -- the core value on screen
6. Portfolio allocation pie chart -- immediate visual insight

**Phase 2** -- Add depth and intelligence:

7. Historical net worth tracking with trend charts -- requires snapshot persistence
8. Auto-polling sync schedule -- automation of Phase 1 manual refresh
9. Emergency fund tracking with status indicators -- first "smart" feature
10. Trading Bot account tagging -- separates active from passive crypto

**Phase 3** -- Advisory intelligence:

11. Macro-level rebalancing suggestions -- target allocation + drift calculation
12. Liquidity sub-allocation -- cash management targets
13. Intra-category rebalancing -- granular targets within categories
14. Allocation drift indicators on dashboard -- visual feedback layer
15. Per-asset cost basis and unrealized P&L -- investment performance visibility

**Defer indefinitely:** Trade execution, multi-user auth, push notifications, mobile native, tax reporting.

## Feature Complexity Matrix

| Feature | Backend Effort | Frontend Effort | Data Model | External API | Risk |
|---------|---------------|-----------------|------------|-------------|------|
| Manual asset CRUD | Low | Low | Low | None | Low |
| CCXT balance import | Medium | Low | Low | CCXT | Medium (API key security, rate limits) |
| Market price fetching | Medium | Low | Low | yahoo-finance2 | Medium (API reliability, rate limits) |
| EUR currency conversion | Low | None | Low | yahoo-finance2 forex | Low |
| Net worth dashboard | Low | Medium | None | None | Low |
| Allocation pie chart | None | Low | None | None | Low |
| Historical snapshots | Medium | Low | Medium | None | Low |
| Trend line chart | Low | Medium | None | None | Low |
| Auto-polling schedule | Low | Low | None | None | Low |
| Trading Bot tagging | Low | Low | Low | None | Low |
| Emergency fund tracking | Low | Medium | Low | None | Low |
| Macro rebalancing | Medium | Medium | Medium | None | Medium (math correctness) |
| Liquidity rebalancing | Medium | Medium | Low | None | Low |
| Intra-category rebalancing | High | Medium | Medium | None | High (complex target configs) |
| Drift indicators | Low | Medium | None | None | Low |
| Cost basis / P&L | Low | Medium | Low | None | Low |

## Competitor Feature Reference

Features observed in comparable products (used to validate table stakes / differentiator classification):

**Ghostfolio** (open-source, NestJS + PostgreSQL):
- Multi-account transaction management
- Portfolio performance (time-weighted return)
- Asset allocation visualization
- Static analysis (fire, impoted, risk identification)
- Import/export transactions
- Dark mode
- PWA with mobile-first design
- No rebalancing suggestions (differentiator for us)

**Sharesight** (commercial):
- Automatic price tracking for 700K+ global instruments
- Dividend tracking and projection
- Diversity report
- Asset allocation visualization
- Cash account and property tracking
- Tax reporting (we explicitly exclude this)

Key insight: Rebalancing suggestions at multiple levels is genuinely uncommon in self-hosted tools. Ghostfolio has allocation views but no rebalancing engine. This confirms multi-level rebalancing as a real differentiator, not just a nice-to-have.

## Sources

- CCXT library documentation (Context7, npmjs.com/package/ccxt) -- fetchBalance, fetchTicker, enableRateLimit
- yahoo-finance2 documentation (Context7, npmjs.com/package/yahoo-finance2) -- quote, historical, forex pairs
- NestJS scheduling documentation (Context7, docs.nestjs.com) -- @Cron decorator, TaskModule
- Recharts documentation (Context7, recharts.org) -- PieChart, LineChart, ResponsiveContainer
- Ghostfolio GitHub README (github.com/ghostfolio/ghostfolio) -- feature inventory, tech stack
- Sharesight features page (sharesight.com/features) -- commercial competitor feature list
- PROJECT.md requirements and constraints -- project scope definition
