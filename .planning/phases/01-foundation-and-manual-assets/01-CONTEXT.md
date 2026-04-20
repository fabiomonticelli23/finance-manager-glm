# Phase 1: Foundation and Manual Assets - Context

**Gathered:** 2026-04-20
**Status:** Ready for planning

<domain>
## Phase Boundary

Project scaffolding, data model with monetary precision, Docker setup, and full CRUD for manual asset management. Users can add, edit, delete manual assets (bank accounts, stocks, ETFs, ETPs), assign them to macro categories, and filter by category. The entire application starts with a single `docker compose up` command.

Requirements covered: DATA-01, DATA-02, DATA-07.

</domain>

<decisions>
## Implementation Decisions

### Asset Form Design
- **D-01:** Dynamic single form — fields appear/hide based on selected asset type (e.g., quantity shows for stocks, balance for bank accounts)
- **D-02:** Minimal required fields (name, type, quantity, currency, category) with optional fields (symbol, institution, purchase price, notes) — user fills what's relevant
- **D-03:** 4 fixed macro categories as enum: Crypto, Stocks, Liquidity, Trading Bots — not user-configurable
- **D-04:** Add/edit form appears as a modal/drawer overlay — contextual, no navigation away from asset list

### Asset List & Browsing
- **D-05:** Table layout for asset list — sortable columns (name, type, quantity, currency, category, value)
- **D-06:** Category tabs across the top: All | Crypto | Stocks | Liquidity | Trading Bots — one-click filtering
- **D-07:** Row-level action buttons for Edit (opens modal) and Delete (shows confirmation dialog)

### App Shell & Navigation
- **D-08:** Left sidebar navigation — standard dashboard pattern
- **D-09:** All future pages shown in sidebar with "Coming soon" placeholder for non-Phase-1 pages (Dashboard, Exchanges, Allocation, History)
- **D-10:** Light theme only in Phase 1 — dark theme deferred
- **D-11:** Prompted empty state — friendly message "No assets yet. Add your first asset to get started." with prominent Add button

### Dev Workflow & Project Structure
- **D-12:** Separate `backend/` and `frontend/` directories with own package.json, Docker config at root
- **D-13:** Hot-reload for development — NestJS CLI for backend, Vite dev server for frontend
- **D-14:** Dockerized PostgreSQL for dev, backend runs locally for hot-reload, TypeORM migrations for schema management

### Claude's Discretion
- Exact modal/drawer component implementation (shadcn/ui Sheet vs Dialog)
- Table column ordering and default sort
- Confirmation dialog copy and styling
- "Coming soon" placeholder page design
- Error state handling for CRUD operations
- Loading skeleton design
- TypeScript strict mode configuration details

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Architecture & Data Model
- `.planning/research/ARCHITECTURE.md` — Full system architecture, entity model (Asset entity, AssetType enum, field definitions), API surface (CRUD endpoints), project directory structure, anti-patterns to avoid, DECIMAL(20,8) convention
- `.planning/REQUIREMENTS.md` — DATA-01 (add asset), DATA-02 (edit/delete asset), DATA-07 (categorize and filter) with acceptance criteria
- `.planning/PROJECT.md` — Tech stack (NestJS 11, React 19, TypeORM 0.3, PostgreSQL, Vite 8, Tailwind 4, shadcn/ui), constraints, key decisions

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- Greenfield project — no existing source code

### Established Patterns
- ARCHITECTURE.md defines module-per-domain pattern for NestJS backend
- shadcn/ui component library for frontend (React 19 compatible)
- TypeORM with migrations (not synchronize) for database schema

### Integration Points
- First phase — establishes patterns all future phases build on
- Asset entity is the core data model that Exchanges (Phase 2) and Portfolio (Phase 3/4) depend on
- API surface at `/api/assets` establishes REST conventions for all future endpoints

</code_context>

<specifics>
## Specific Ideas

- Form should adapt intelligently: bank account types show institution + balance fields, stock/ETF types show symbol + quantity fields
- Table should feel scannable — dense data but not cluttered
- Modal/drawer should keep users in context of the asset list

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-foundation-and-manual-assets*
*Context gathered: 2026-04-20*
