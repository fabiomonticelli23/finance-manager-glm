<!-- GSD:project-start source:PROJECT.md -->
## Project

Finance Manager — A personal portfolio management web application that aggregates all assets and investments into a single dashboard. Connects to crypto exchanges via CCXT, tracks manual assets (bank accounts, stocks, ETFs, ETPs) with auto-fetched market prices, and provides total net worth visibility, portfolio allocation analytics, and multi-level rebalancing suggestions. Built for a single user.

**Core Value:** Complete visibility into total net worth across all asset classes, converted to EUR, with actionable rebalancing guidance.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:STACK.md -->
## Technology Stack

**Backend:** NestJS 11, TypeORM 0.3, PostgreSQL, CCXT 4.5 (crypto exchanges), yahoo-finance2 (market prices)
**Frontend:** React 19, Vite 8, Tailwind 4, shadcn/ui, Recharts 3 (charts)
**Infrastructure:** Docker (3 Dockerfiles + docker-compose), Node.js 22 LTS
**Key decisions:** TypeORM over Prisma (tighter NestJS integration), Recharts over Chart.js (React 19 support), DECIMAL(20,8) for all monetary values
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Six backend modules: Assets, Exchanges, Sync, MarketData, Portfolio, Dashboard. One-directional data flow: external APIs write to DB via scheduled sync, all reads serve from DB only. Single REST API surface (~16 endpoints). API keys encrypted at rest (AES-256).

See `.planning/research/ARCHITECTURE.md` for full details.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, or `.github/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` — do not edit manually.
<!-- GSD:profile-end -->
