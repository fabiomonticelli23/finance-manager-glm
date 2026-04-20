# Requirements: Finance Manager

**Defined:** 2026-04-20
**Core Value:** Complete visibility into total net worth across all asset classes, converted to EUR, with actionable rebalancing guidance.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Data Layer

- [ ] **DATA-01**: User can add manual assets (bank accounts, current accounts, deposit accounts, stocks, ETFs, ETPs, non-CCXT exchanges) with name, type, quantity, currency, and institution
- [ ] **DATA-02**: User can edit and delete manual assets
- [ ] **DATA-03**: User can connect crypto exchanges via CCXT with API keys and import balances automatically
- [ ] **DATA-04**: User can disconnect crypto exchange connections
- [ ] **DATA-05**: System auto-fetches current market prices for stocks, ETFs, and ETPs via market data API
- [ ] **DATA-06**: System converts all asset values to EUR base currency using current FX rates
- [ ] **DATA-07**: User can categorize assets into macro categories (Crypto, Stocks, Liquidity, Trading Bots)

### Dashboard

- [ ] **DASH-01**: User can see total net worth across all assets in EUR on a single dashboard
- [ ] **DASH-02**: User can see portfolio allocation by macro category in a pie chart
- [ ] **DASH-03**: User can see historical net worth trend over time with a line chart
- [ ] **DASH-04**: User can see allocation drift indicators when portfolio deviates from target allocations

### Sync

- [ ] **SYNC-01**: System auto-polls CCXT exchanges and market prices on a configurable schedule
- [ ] **SYNC-02**: User can trigger manual data refresh on demand

### Rebalancing

- [ ] **REBAL-01**: User can set target allocation percentages at macro level (Crypto, Stocks, Liquidity, Trading Bots)
- [ ] **REBAL-02**: User can see macro-level rebalancing suggestions based on current vs target allocation
- [ ] **REBAL-03**: User can set target allocation within Liquidity category (current accounts, deposit accounts, emergency fund)
- [ ] **REBAL-04**: User can see liquidity-level rebalancing suggestions
- [ ] **REBAL-05**: User can set target allocation within each macro category (e.g., BTC vs ETH within Crypto)
- [ ] **REBAL-06**: User can see intra-category rebalancing suggestions
- [ ] **REBAL-07**: User can set emergency fund target (value or percentage) protected from rebalancing suggestions
- [ ] **REBAL-08**: System highlights emergency fund status (underfunded, on target, overfunded) on dashboard

### Trading Bots

- [ ] **TBOT-01**: User can tag CCXT-connected accounts as "Trading Bot" to group them under Trading Bots macro category

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Cost Basis

- **COST-01**: User can track purchase price per asset
- **COST-02**: User can see unrealized P&L per asset

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Trade execution | Security liability; rebalancing is advisory only |
| Multi-user authentication | Solo user personal tool |
| Push/email notifications | Dashboard indicators suffice |
| Mobile native app | Web-first, responsive design |
| Tax reporting | Jurisdiction-specific, out of scope |
| Real-time streaming prices | Polling-based refresh is sufficient |
| Social features | Private financial tool |
| News feed | Scope creep into financial media |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| DATA-01 | Phase 1 | Pending |
| DATA-02 | Phase 1 | Pending |
| DATA-03 | Phase 2 | Pending |
| DATA-04 | Phase 2 | Pending |
| DATA-05 | Phase 2 | Pending |
| DATA-06 | Phase 2 | Pending |
| DATA-07 | Phase 1 | Pending |
| DASH-01 | Phase 3 | Pending |
| DASH-02 | Phase 3 | Pending |
| DASH-03 | Phase 3 | Pending |
| DASH-04 | Phase 3 | Pending |
| SYNC-01 | Phase 2 | Pending |
| SYNC-02 | Phase 2 | Pending |
| REBAL-01 | Phase 4 | Pending |
| REBAL-02 | Phase 4 | Pending |
| REBAL-03 | Phase 4 | Pending |
| REBAL-04 | Phase 4 | Pending |
| REBAL-05 | Phase 4 | Pending |
| REBAL-06 | Phase 4 | Pending |
| REBAL-07 | Phase 4 | Pending |
| REBAL-08 | Phase 4 | Pending |
| TBOT-01 | Phase 2 | Pending |

**Coverage:**
- v1 requirements: 22 total
- Mapped to phases: 22
- Unmapped: 0

---
*Requirements defined: 2026-04-20*
*Last updated: 2026-04-20 after roadmap creation*
