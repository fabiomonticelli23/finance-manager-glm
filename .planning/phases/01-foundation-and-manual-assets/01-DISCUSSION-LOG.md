# Phase 1: Foundation and Manual Assets - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-20
**Phase:** 01-foundation-and-manual-assets
**Areas discussed:** Asset Form Design, Asset List & Browsing, App Shell & Navigation, Dev Workflow & Project Structure

---

## Asset Form Design

| Option | Description | Selected |
|--------|-------------|----------|
| Dynamic single form | Fields appear/hide based on asset type | Yes |
| Separate forms per type | Each asset type gets its own form | |
| Flat form with all fields | One standard form, every field always visible | |

**User's choice:** Dynamic single form
**Notes:** Simpler navigation, less clicking around

| Option | Description | Selected |
|--------|-------------|----------|
| Minimal required, rest optional | Name, type, quantity, currency required; symbol, institution, purchase price optional | Yes |
| All fields always visible | Every field shown, user decides what to fill | |
| Bare minimum required | Only name, type, category required | |

**User's choice:** Minimal required, rest optional

| Option | Description | Selected |
|--------|-------------|----------|
| Fixed 4 categories | Hardcoded enum — Crypto, Stocks, Liquidity, Trading Bots | Yes |
| User-configurable categories | Database-stored, user can add/edit | |

**User's choice:** Fixed 4 categories

| Option | Description | Selected |
|--------|-------------|----------|
| Modal/drawer | Overlay form, contextual, no navigation | Yes |
| Separate creation page | Dedicated page, then redirect back | |
| Inline in list | Empty row appears in table | |

**User's choice:** Modal/drawer

---

## Asset List & Browsing

| Option | Description | Selected |
|--------|-------------|----------|
| Table | Columns for name, type, quantity, currency, category. Sortable, scannable. | Yes |
| Cards | Each asset as a card showing key info | |
| Responsive hybrid | Table desktop, cards mobile | |

**User's choice:** Table

| Option | Description | Selected |
|--------|-------------|----------|
| Category tabs | All/Crypto/Stocks/Liquidity/Trading Bots tabs across top | Yes |
| Dropdown + search | Multi-select filter plus search bar | |
| Search only | Simple text filter | |

**User's choice:** Category tabs

| Option | Description | Selected |
|--------|-------------|----------|
| Row action buttons | Edit/Delete icons per row | Yes |
| Click-to-detail | Click row for detail view with actions | |
| Bulk action bar | Checkbox select + bulk actions | |

**User's choice:** Row action buttons

---

## App Shell & Navigation

| Option | Description | Selected |
|--------|-------------|----------|
| Sidebar nav | Left sidebar with nav links | Yes |
| Top navigation bar | Horizontal links at top | |
| Responsive sidebar/bottom tabs | Sidebar desktop, bottom tabs mobile | |

**User's choice:** Sidebar nav

| Option | Description | Selected |
|--------|-------------|----------|
| Assets only + placeholders | Assets works, other pages show "Coming soon" | Yes |
| Assets only, minimal sidebar | Only Assets link in sidebar | |

**User's choice:** Assets only + placeholders

| Option | Description | Selected |
|--------|-------------|----------|
| Light only | Light theme, dark added later | Yes |
| Light + dark | Both themes from start | |

**User's choice:** Light only for now

| Option | Description | Selected |
|--------|-------------|----------|
| Prompted empty state | Friendly message + Add button | Yes |
| Minimal empty state | Empty table with headers | |
| Guided onboarding flow | Step-by-step asset addition | |

**User's choice:** Prompted empty state

---

## Dev Workflow & Project Structure

| Option | Description | Selected |
|--------|-------------|----------|
| Separate dirs | backend/ and frontend/ with own package.json | Yes |
| Monorepo with workspaces | Single package.json with workspaces | |

**User's choice:** Separate directories

| Option | Description | Selected |
|--------|-------------|----------|
| Hot-reload for both | NestJS CLI + Vite dev server | Yes |
| Docker-only | Production-style, rebuild on change | |

**User's choice:** Hot-reload for both

| Option | Description | Selected |
|--------|-------------|----------|
| Dockerized Postgres + migrations | Postgres in Docker, backend local for dev | Yes |
| Full Docker | Everything in Docker including backend | |

**User's choice:** Dockerized Postgres + migrations

---

## Claude's Discretion

- Modal/drawer component choice (shadcn/ui Sheet vs Dialog)
- Table column ordering and default sort
- Confirmation dialog copy and styling
- "Coming soon" placeholder page design
- Error state handling for CRUD operations
- Loading skeleton design

## Deferred Ideas

None — discussion stayed within phase scope
