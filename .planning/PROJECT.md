# Finance Manager

## What This Is

A personal portfolio management web application that aggregates all assets and investments into a single dashboard. Connects to crypto exchanges via CCXT, tracks manual assets (bank accounts, stocks, ETFs, ETPs) with auto-fetched market prices, and provides total net worth visibility, portfolio allocation analytics, and multi-level rebalancing suggestions. Built for a single user.

## Core Value

Complete visibility into total net worth across all asset classes, converted to EUR, with actionable rebalancing guidance.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Aggregate crypto exchange data via CCXT library with configurable API keys
- [ ] Track manual assets: bank accounts (current, deposit), emergency fund, stocks, ETFs, ETPs, non-CCXT exchanges
- [ ] Auto-fetch market prices for stocks, ETFs, and ETPs via market data API
- [ ] Display total net worth in EUR as base currency with multi-currency conversion
- [ ] Portfolio allocation dashboard with pie charts and bar charts by macro category (Crypto, Stocks, Liquidity, Trading Bots)
- [ ] Multi-level rebalancing suggestions:
  - Macro level: target allocation across Crypto, Stocks, Liquidity, Trading Bots
  - Liquidity level: target allocation between current accounts, deposit accounts, emergency fund
  - Intra-category level: target allocation within each category (e.g., BTC vs ETH vs altcoins)
- [ ] Emergency fund treated as protected allocation with target value/percentage, highlighted when underfunded or overfunded
- [ ] CCXT accounts tagged as "Trading Bot" grouped under separate Trading Bots macro category
- [ ] CCXT data sync via auto-polling on schedule + manual refresh on demand
- [ ] Historical net worth tracking with trend charts over time
- [ ] Dashboard-only indicators for allocation drift and emergency fund status (no push/email alerts)
- [ ] Fully containerized with Docker (backend, frontend, database Dockerfiles + docker-compose.yml)

### Out of Scope

- Multi-user support — single user personal tool
- Trade execution via the app — rebalancing is advisory only
- Push notifications or email alerts — dashboard indicators only
- Mobile native app — web-first
- OAuth / social login — solo user, no authentication system needed

## Context

- User manages a diversified portfolio across crypto exchanges, traditional brokerage (stocks/ETFs/ETPs), and bank accounts (current and deposit)
- Crypto exchange connections use CCXT library which standardizes API access across 100+ exchanges
- Market data API needed for real-time stock/ETF/ETP pricing (e.g., Yahoo Finance, Alpha Vantage, or similar)
- Trading bots operate on CCXT-connected exchanges; these accounts are tagged separately from passive crypto holdings
- Euro (EUR) is the base currency; FX conversion needed for USD-denominated and crypto assets
- The app needs to persist historical snapshots to render net worth trend lines

## Constraints

- **Tech Stack**: NestJS backend, React frontend, PostgreSQL database — user-specified
- **Containerization**: Docker with separate Dockerfiles per service and docker-compose orchestration
- **CCXT Integration**: Limited to exchanges supported by CCXT library
- **Data Privacy**: API keys and financial data stored locally; no external data sharing

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Solo user, no auth | Single-user personal tool, no multi-tenancy needed | — Pending |
| Advisory-only rebalancing | App suggests changes, user executes manually outside the app | — Pending |
| CCXT for crypto exchange data | Standardized API across 100+ exchanges, well-maintained library | — Pending |
| Auto-fetch prices for tradable assets | Stocks/ETFs/ETPs need current market prices; manual entry is error-prone | — Pending |
| Trading Bots as tagged CCXT accounts | Trading bot accounts are CCXT connections flagged with a tag, not a separate data source | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-20 after initialization*
