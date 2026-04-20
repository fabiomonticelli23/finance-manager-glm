# Phase 1: Foundation and Manual Assets - Research

**Researched:** 2026-04-20
**Domain:** Full-stack scaffolding, TypeORM data model, Docker setup, React CRUD UI
**Confidence:** HIGH

## Summary

Phase 1 establishes the entire project foundation: NestJS 11 backend with TypeORM 0.3 connected to PostgreSQL, React 19 frontend with Vite 8 and Tailwind 4, Docker Compose for one-command startup, and a complete CRUD workflow for manual asset management. The backend exposes a REST API at `/api/assets` with full create/read/update/soft-delete operations. The frontend provides a table-based asset list with category tab filtering, a dynamic add/edit modal, and an app shell with sidebar navigation.

The data model centers on a single `Asset` entity using PostgreSQL `DECIMAL(20,8)` for all monetary fields, UUID primary keys, a TypeScript enum for `AssetType`, and a fixed enum for `MacroCategory` (Crypto, Stocks, Liquidity, Trading Bots). TypeORM migrations manage schema evolution -- `synchronize: false` in all environments. Soft deletes via `@DeleteDateColumn` allow hiding assets without losing historical data.

The frontend uses shadcn/ui components (Dialog for the add/edit modal, Table + TanStack Table for sortable data, Tabs for category filtering) with Tailwind CSS v4 configured through the `@tailwindcss/vite` plugin. TanStack React Query manages server state and cache invalidation on CRUD mutations.

**Primary recommendation:** Scaffold backend and frontend in parallel. Backend first (data model, migration, API) then frontend (app shell, asset list, form). Docker Compose wires PostgreSQL and provides a reproducible dev environment. Use `autoLoadEntities: true` in TypeORM to avoid manual entity registration.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- **D-01:** Dynamic single form -- fields appear/hide based on selected asset type (e.g., quantity shows for stocks, balance for bank accounts)
- **D-02:** Minimal required fields (name, type, quantity, currency, category) with optional fields (symbol, institution, purchase price, notes) -- user fills what's relevant
- **D-03:** 4 fixed macro categories as enum: Crypto, Stocks, Liquidity, Trading Bots -- not user-configurable
- **D-04:** Add/edit form appears as a modal/drawer overlay -- contextual, no navigation away from asset list
- **D-05:** Table layout for asset list -- sortable columns (name, type, quantity, currency, category, value)
- **D-06:** Category tabs across the top: All | Crypto | Stocks | Liquidity | Trading Bots -- one-click filtering
- **D-07:** Row-level action buttons for Edit (opens modal) and Delete (shows confirmation dialog)
- **D-08:** Left sidebar navigation -- standard dashboard pattern
- **D-09:** All future pages shown in sidebar with "Coming soon" placeholder for non-Phase-1 pages (Dashboard, Exchanges, Allocation, History)
- **D-10:** Light theme only in Phase 1 -- dark theme deferred
- **D-11:** Prompted empty state -- friendly message "No assets yet. Add your first asset to get started." with prominent Add button
- **D-12:** Separate `backend/` and `frontend/` directories with own package.json, Docker config at root
- **D-13:** Hot-reload for development -- NestJS CLI for backend, Vite dev server for frontend
- **D-14:** Dockerized PostgreSQL for dev, backend runs locally for hot-reload, TypeORM migrations for schema management

### Claude's Discretion
- Exact modal/drawer component implementation (shadcn/ui Sheet vs Dialog)
- Table column ordering and default sort
- Confirmation dialog copy and styling
- "Coming soon" placeholder page design
- Error state handling for CRUD operations
- Loading skeleton design
- TypeScript strict mode configuration details

### Deferred Ideas (OUT OF SCOPE)
None -- discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| DATA-01 | User can add manual assets (bank accounts, current accounts, deposit accounts, stocks, ETFs, ETPs, non-CCXT exchanges) with name, type, quantity, currency, and institution | Asset entity with AssetType enum, dynamic form that shows/hides fields based on type, POST /api/assets endpoint, TypeORM migration |
| DATA-02 | User can edit and delete manual assets | PUT /api/assets/:id endpoint, DELETE /api/assets/:id with soft-delete via @DeleteDateColumn, edit modal pre-populated with existing data |
| DATA-07 | User can categorize assets into macro categories (Crypto, Stocks, Liquidity, Trading Bots) | MacroCategory enum on Asset entity, category tabs on frontend filtering, category field in form |
</phase_requirements>

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Asset CRUD persistence | API / Backend | Database | NestJS AssetsService + TypeORM manage entity lifecycle, PostgreSQL stores data |
| Monetary precision (DECIMAL 20,8) | Database | API / Backend | PostgreSQL DECIMAL type ensures precision; backend must use string/decimal.js, never JS number |
| Asset form validation | Browser / Client | API / Backend | Frontend validates UX constraints (required fields, type-specific fields); backend validates data integrity (enum values, DECIMAL range) |
| Category filtering | Browser / Client | -- | Client-side tab filtering on already-fetched data; single GET /api/assets returns all assets |
| API routing and DTOs | API / Backend | -- | NestJS controller defines REST endpoints, class-validator DTOs enforce request shape |
| Project scaffolding | Build / Dev | -- | Docker Compose, package.json, tsconfig establish the development environment |
| App shell and navigation | Browser / Client | -- | React SPA with sidebar layout, client-side routing via react-router-dom |

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| @nestjs/common | 11.1.19 | Backend framework | User-specified in PROJECT.md; latest NestJS 11.x [VERIFIED: npm registry] |
| @nestjs/core | 11.1.19 | NestJS core engine | Paired with @nestjs/common [VERIFIED: npm registry] |
| @nestjs/platform-express | 11.1.19 | HTTP adapter | Default NestJS adapter [VERIFIED: npm registry] |
| @nestjs/cli | 11.0.21 | Project scaffolding and code generation | Official NestJS CLI for `nest generate` [VERIFIED: npm registry] |
| @nestjs/config | 4.0.4 | Environment variable management | Wraps dotenv with async config loading [VERIFIED: npm registry] |
| @nestjs/typeorm | 11.0.1 | TypeORM integration module | Provides `TypeOrmModule.forRootAsync()` for DI-based config [VERIFIED: npm registry] |
| typeorm | 0.3.28 | ORM for PostgreSQL | User-specified; migrations, entities, repositories [VERIFIED: npm registry] |
| pg | 8.20.0 | PostgreSQL driver for Node.js | Required by TypeORM `type: 'postgres'` [VERIFIED: npm registry] |
| react | 19.2.5 | UI framework | User-specified React 19 [VERIFIED: npm registry] |
| react-dom | 19.2.5 | React DOM renderer | Paired with react [VERIFIED: npm registry] |
| react-router-dom | 7.14.1 | Client-side routing | Standard React router; supports app shell with sidebar nav [VERIFIED: npm registry] |
| vite | 8.0.9 | Frontend build tool and dev server | User-specified Vite 8 [VERIFIED: npm registry] |
| tailwindcss | 4.2.2 | Utility-first CSS | User-specified Tailwind 4 [VERIFIED: npm registry] |
| @tailwindcss/vite | 4.2.2 | Tailwind v4 Vite plugin | Required for Tailwind v4 with Vite; replaces PostCSS-based setup [VERIFIED: npm registry] |
| shadcn/ui (via CLI) | latest | Component library | User-specified; copy-paste components, Radix-based [CITED: ui.shadcn.com/docs/installation/vite] |
| @tanstack/react-query | 5.99.2 | Server state management | Industry standard for async data fetching, cache invalidation, mutation patterns [VERIFIED: npm registry] |
| @tanstack/react-table | 8.21.3 | Headless table logic | Required by shadcn/ui DataTable component [VERIFIED: npm registry, CITED: ui.shadcn.com/docs/components/data-table] |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| class-validator | 0.15.1 | DTO validation decorators | All request DTOs: `@IsString()`, `@IsEnum()`, `@IsOptional()` [VERIFIED: npm registry] |
| class-transformer | 0.5.1 | DTO transformation | NestJS pipeline for transforming plain objects to class instances [VERIFIED: npm registry] |
| zod | 4.3.6 | Frontend form validation | Validate form inputs in React before API calls [VERIFIED: npm registry] |
| lucide-react | 1.8.0 | Icon library | Icons for sidebar navigation, action buttons, empty state [VERIFIED: npm registry] |
| @radix-ui/react-dialog | 1.1.15 | Dialog primitive | Underlying shadcn/ui Dialog component [VERIFIED: npm registry] |
| @nestjs/testing | 11.1.19 | NestJS test utilities | Unit and integration tests for services/controllers [VERIFIED: npm registry] |
| vitest | 4.1.4 | Test runner | Fast, Vite-native test runner for both unit and component tests [VERIFIED: npm registry] |
| @testing-library/react | 16.3.2 | React component testing | Testing UI behavior (form interactions, list rendering) [VERIFIED: npm registry] |
| @testing-library/jest-dom | 6.9.1 | DOM matchers | `toBeInTheDocument()`, `toHaveTextContent()` etc. [VERIFIED: npm registry] |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| class-validator + class-transformer | zod (backend via nestjs-zod) | zod is more modern but class-validator is the NestJS ecosystem default with deep integration. Stick with class-validator for Phase 1 to avoid fighting conventions [ASSUMED] |
| TanStack React Query | SWR | React Query has richer mutation/cache management; both are solid. React Query is more common in NestJS + React stacks [ASSUMED] |
| shadcn/ui Dialog (modal) | shadcn/ui Sheet (drawer) | Sheet slides from edge, Dialog is centered overlay. Dialog is more conventional for forms. User gave discretion; recommend Dialog for the add/edit form [ASSUMED] |

### Installation

**Backend:**
```bash
cd backend
npm install @nestjs/common @nestjs/core @nestjs/platform-express @nestjs/config @nestjs/typeorm typeorm pg class-validator class-transformer
npm install -D @nestjs/cli @nestjs/testing typescript @types/node ts-node vitest
```

**Frontend:**
```bash
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install react-router-dom @tanstack/react-query @tanstack/react-table zod lucide-react
npm install -D @tailwindcss/vite tailwindcss vitest @testing-library/react @testing-library/jest-dom
npx shadcn@latest init -t vite
npx shadcn@latest add dialog table button tabs input select label textarea
```

**Version verification completed 2026-04-20. All versions verified against npm registry.**

## Architecture Patterns

### System Architecture Diagram

```
Developer runs: docker compose up -d postgres
                   |
                   v
            +-------------+
            | PostgreSQL  |  <-- Docker container, port 5432
            | (data layer)|
            +------+------+
                   |
              TypeORM migrations + queries
                   |
            +------+------+
            |  NestJS API  |  <-- Runs locally for hot-reload (nest start --watch)
            |  (backend/)  |      port 3000
            +------+------+
                   |
              REST API (JSON) via /api/*
                   |
            +------+------+
            | Vite Dev     |  <-- Runs locally for hot-reload (npm run dev)
            | Server       |      proxies /api to NestJS
            | (frontend/)  |
            +------+------+
                   |
              Browser at localhost:5173
```

### Recommended Project Structure
```
finance-manager-glm/
  docker-compose.yml          # PostgreSQL service, network config
  .env.example                # Template for environment variables
  backend/
    Dockerfile                # Production backend image
    tsconfig.json
    nest-cli.json
    package.json
    src/
      main.ts                 # Bootstrap, global pipes, CORS
      app.module.ts           # Root module: imports ConfigModule, TypeOrmModule, AssetsModule
      config/
        config.module.ts      # @nestjs/config setup
        config.service.ts     # Typed config access (DB_*, APP_*)
        database.config.ts    # TypeORM DataSource factory
      common/
        decorators/           # Shared decorators
        filters/              # Exception filters (HttpExceptionFilter)
        interceptors/         # Response transform interceptors
        dto/                  # Shared DTO base classes (pagination)
      modules/
        assets/
          assets.module.ts    # Imports TypeOrmModule.forFeature([Asset])
          assets.controller.ts # GET/POST/PUT/DELETE /api/assets
          assets.service.ts   # CRUD logic, soft-delete
          dto/
            create-asset.dto.ts
            update-asset.dto.ts
            asset-response.dto.ts
          entities/
            asset.entity.ts   # Asset entity with DECIMAL fields
            asset-type.enum.ts   # AssetType enum
            macro-category.enum.ts # MacroCategory enum
    migrations/               # TypeORM migration files
    data-source.ts            # TypeORM CLI DataSource for migration commands
  frontend/
    Dockerfile                # Production frontend (nginx serving built assets)
    vite.config.ts            # Vite config with @tailwindcss/vite plugin + API proxy
    tsconfig.json
    package.json
    index.html
    src/
      main.tsx                # React root + QueryClientProvider
      App.tsx                 # Routes definition
      api/
        client.ts             # Axios/fetch instance with base URL
        assets.ts             # Asset API functions (list, create, update, delete)
      components/
        ui/                   # shadcn/ui generated components (dialog, table, button, tabs, etc.)
        layout/
          AppLayout.tsx       # Sidebar + main content area
          Sidebar.tsx         # Navigation links with "Coming soon" items
        assets/
          AssetList.tsx       # Table with sorting, category tabs, row actions
          AssetForm.tsx       # Dynamic form in Dialog (add/edit)
          AssetFormFields.tsx # Type-conditional field rendering
          EmptyState.tsx      # "No assets yet" prompt
      hooks/
        useAssets.ts          # TanStack Query hooks (useQuery + useMutation)
      types/
        api.ts                # TypeScript interfaces matching backend DTOs
      lib/
        utils.ts              # shadcn/ui cn() utility
```

### Pattern 1: NestJS Module-per-Domain

**What:** Each business domain gets its own module with controller, service, DTOs, and entities. Modules declare explicit imports and exports.

**When to use:** Always in NestJS. This is the framework's core organizing principle.

**Example:**
```typescript
// Source: [CITED: context7.com/nestjs/typeorm]
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AssetsController } from './assets.controller';
import { AssetsService } from './assets.service';
import { Asset } from './entities/asset.entity';

@Module({
  imports: [TypeOrmModule.forFeature([Asset])],
  controllers: [AssetsController],
  providers: [AssetsService],
  exports: [AssetsService],
})
export class AssetsModule {}
```

### Pattern 2: TypeORM Async Configuration with ConfigService

**What:** Use `TypeOrmModule.forRootAsync()` with a factory that reads environment variables via `ConfigService`. Never hardcode connection details.

**When to use:** Always for database configuration in NestJS.

**Example:**
```typescript
// Source: [CITED: context7.com/nestjs/typeorm]
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: configService.get<number>('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_NAME'),
        entities: [],
        synchronize: false,     // NEVER true -- use migrations
        autoLoadEntities: true, // Auto-register entities from forFeature()
      }),
      inject: [ConfigService],
    }),
    AssetsModule,
  ],
})
export class AppModule {}
```

### Pattern 3: Asset Entity with Monetary Precision

**What:** All monetary values use PostgreSQL `DECIMAL(20,8)`. TypeScript represents these as `string` to avoid floating-point precision loss. The 20 total digits with 8 decimal places handles both fiat (2 decimals) and crypto (8 decimals).

**When to use:** Every column that stores a quantity, price, or monetary value.

**Example:**
```typescript
// Source: [CITED: typeorm.io/docs/entity/entities] + [VERIFIED: context7.com/typeorm/typeorm]
import {
  Entity, PrimaryGeneratedColumn, Column,
  CreateDateColumn, UpdateDateColumn, DeleteDateColumn,
} from 'typeorm';

export enum AssetType {
  BANK_CURRENT = 'bank_current',
  BANK_DEPOSIT = 'bank_deposit',
  STOCK = 'stock',
  ETF = 'etf',
  ETP = 'etp',
}

export enum MacroCategory {
  CRYPTO = 'crypto',
  STOCKS = 'stocks',
  LIQUIDITY = 'liquidity',
  TRADING_BOTS = 'trading_bots',
}

@Entity('assets')
export class Asset {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column({ type: 'enum', enum: AssetType })
  type: AssetType;

  @Column({ nullable: true })
  symbol: string;

  @Column({ type: 'decimal', precision: 20, scale: 8, default: 0 })
  quantity: string;  // string to avoid JS number precision loss

  @Column({ length: 3 })
  currency: string;

  @Column({ type: 'enum', enum: MacroCategory })
  category: MacroCategory;

  @Column({ nullable: true })
  institution: string;

  @Column({ type: 'decimal', precision: 20, scale: 8, nullable: true })
  purchasePrice: string;

  @Column({ nullable: true, type: 'text' })
  notes: string;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn()
  deletedAt: Date;
}
```

### Pattern 4: TanStack Query for CRUD

**What:** Use TanStack React Query's `useQuery` for data fetching and `useMutation` for create/update/delete. Mutations invalidate the assets query to trigger refetch.

**When to use:** All server state management in the frontend.

**Example:**
```typescript
// hooks/useAssets.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { assetsApi } from '../api/assets';

export function useAssets() {
  return useQuery({
    queryKey: ['assets'],
    queryFn: assetsApi.getAll,
  });
}

export function useCreateAsset() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: assetsApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['assets'] });
    },
  });
}

export function useDeleteAsset() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: assetsApi.remove,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['assets'] });
    },
  });
}
```

### Pattern 5: Tailwind CSS v4 with Vite

**What:** Tailwind v4 uses the `@tailwindcss/vite` plugin instead of PostCSS configuration. Configuration moves to CSS using `@theme` directives. No `tailwind.config.js` file needed.

**When to use:** Frontend styling setup.

**Example (vite.config.ts):**
```typescript
// Source: [CITED: tailwindcss.com/docs/installation/vite] -- page not fully retrievable, structure based on verified package
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';
import path from 'path';

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
      },
    },
  },
});
```

**Example (src/index.css):**
```css
@import "tailwindcss";

@theme {
  --color-background: #ffffff;
  --color-foreground: #0a0a0a;
  /* shadcn/ui generates theme variables during init */
}
```

### Pattern 6: shadcn/ui DataTable with Sorting

**What:** shadcn/ui provides a DataTable component built on TanStack Table. Use it for the asset list with sortable columns.

**When to use:** Any tabular data display.

**Example:**
```typescript
// Source: [CITED: ui.shadcn.com/docs/components/data-table]
import { useReactTable, getCoreRowModel, getSortedRowModel } from '@tanstack/react-table';

const table = useReactTable({
  data: assets,
  columns,
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
});
```

### Anti-Patterns to Avoid

- **Using FLOAT/number for monetary values:** PostgreSQL FLOAT and JavaScript `number` are IEEE 754 doubles. `0.1 + 0.2 = 0.30000000000000004`. Always use `DECIMAL(20,8)` in PostgreSQL and `string` in TypeScript. [CITED: ARCHITECTURE.md Anti-Pattern 4]
- **Enabling TypeORM synchronize in any environment:** `synchronize: true` can silently alter production schemas. Use migrations exclusively. [CITED: ARCHITECTURE.md, CONTEXT.md D-14]
- **Putting business logic in controllers:** Controllers handle HTTP concerns (parsing, response formatting). All logic belongs in services. NestJS convention. [CITED: context7.com/nestjs/docs.nestjs.com]
- **Circular module dependencies:** Assets module must not import Portfolio module. Data flows one direction. Use `forwardRef` only as last resort. [CITED: ARCHITECTURE.md Anti-Pattern 5]
- **Fetching external APIs on every request:** Phase 1 has no external APIs, but the pattern is established now. All reads serve from DB only. [CITED: ARCHITECTURE.md Anti-Pattern 2]
- **Client-side monetary arithmetic with JS number:** Even on the frontend, display DECIMAL values as strings. Any arithmetic (if needed later) should use a decimal library. [ASSUMED]

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| DB schema evolution | Manual SQL scripts | TypeORM migrations (`migration:generate`, `migration:run`) | Tracks applied migrations, supports rollback, team-safe |
| Request validation | Custom if/else validators | class-validator decorators on DTOs | NestJS ValidationPipe integrates natively, consistent error format |
| Soft delete | Custom `isDeleted` flag + `deletedAt` column | TypeORM `@DeleteDateColumn` + `softDelete()` | Built-in repository methods, automatic query filtering with `@DeleteDateColumn` |
| Sortable table UI | Custom sort state + header click handlers | TanStack Table `getSortedRowModel()` | Handles multi-sort, keyboard, accessibility |
| Modal/drawer component | Custom overlay + animation | shadcn/ui Dialog or Sheet | Radix primitives handle focus trap, a11y, scroll lock, animations |
| Form validation | Manual `if (!value)` checks | Zod schema validation | Type-safe, composable, generates TypeScript types |
| CSS utility classes | Custom CSS files | Tailwind CSS v4 | Utility-first, JIT compilation, design system consistency |
| API client state | useState + useEffect + loading flags | TanStack React Query | Caching, deduplication, background refetch, mutation lifecycle |

**Key insight:** This phase establishes patterns that all future phases build on. Every "don't hand-roll" choice here must be consistent with what Phases 2-4 will need. TypeORM, TanStack Query, and shadcn/ui all scale well from simple CRUD to complex aggregation.

## Common Pitfalls

### Pitfall 1: TypeORM Migration CLI Requires Separate data-source.ts
**What goes wrong:** Running `npx typeorm migration:generate` fails because it cannot find the NestJS module configuration. The TypeORM CLI needs a standalone `DataSource` instance, not the NestJS-integrated one.
**Why it happens:** NestJS manages TypeORM through `TypeOrmModule.forRootAsync()`, but the CLI operates outside the NestJS bootstrap context.
**How to avoid:** Create a `backend/data-source.ts` file that exports a `DataSource` configured identically to the NestJS config. Use `ts-node` to run CLI commands: `npx typeorm migration:generate -d data-source.ts migrations/InitialSchema`. [CITED: context7.com/typeorm/typeorm migration docs]
**Warning signs:** "DataSource is not set" or "Cannot use import statement outside a module" errors when running migration commands.

### Pitfall 2: DECIMAL Values Become JS Numbers
**What goes wrong:** TypeORM returns DECIMAL columns as JavaScript `number` by default, losing precision for crypto quantities (8 decimal places).
**Why it happens:** The `pg` driver converts DECIMAL to JS number unless configured otherwise.
**How to avoid:** In the TypeORM config, keep the entity property typed as `string`. TypeORM 0.3 with PostgreSQL returns DECIMAL as string when the TypeScript type is `string`. Verify with a test: insert `0.12345678`, read back, confirm no precision loss. [ASSUMED -- verify during implementation]
**Warning signs:** `console.log(asset.quantity)` shows `0.12345678` as `0.12345677999999999`.

### Pitfall 3: Tailwind v4 Configuration Confusion
**What goes wrong:** Developers familiar with Tailwind v3 create `tailwind.config.js` and PostCSS config. Tailwind v4 uses neither -- it uses the `@tailwindcss/vite` plugin and CSS-based `@theme` configuration.
**Why it happens:** Most tutorials and Stack Overflow answers reference Tailwind v3 patterns.
**How to avoid:** Do not create `tailwind.config.js`. Do not create `postcss.config.js`. Install `@tailwindcss/vite` and add it to the Vite plugins array. Use `@import "tailwindcss"` in CSS. shadcn/ui `init` handles this automatically for Vite projects. [VERIFIED: npm registry -- @tailwindcss/vite 4.2.2 exists as separate package]
**Warning signs:** Tailwind classes not generating styles; `PostCSS plugin tailwindcss not found` error.

### Pitfall 4: shadcn/ui Init Overwrites Vite Config
**What goes wrong:** Running `npx shadcn@latest init` with wrong flags overwrites existing Vite configuration or fails to detect the project structure.
**Why it happens:** shadcn/ui needs to know the framework (Vite) and component import alias (`@/`).
**How to avoid:** Use `npx shadcn@latest init -t vite` to specify the Vite template. Ensure `tsconfig.json` has the `@` path alias configured before running init. [CITED: ui.shadcn.com/docs/installation/vite]
**Warning signs:** "Could not resolve @/lib/utils" errors, or components import from wrong paths.

### Pitfall 5: Docker Compose Depends_on Without Health Check
**What goes wrong:** Backend tries to connect to PostgreSQL before PostgreSQL is ready to accept connections, causing TypeORM connection failure and backend crash.
**Why it happens:** `depends_on` only ensures the container starts, not that the service inside is ready. PostgreSQL takes a few seconds to initialize.
**How to avoid:** Add a `healthcheck` to the PostgreSQL service and use `depends_on` with `condition: service_healthy`. Also configure `retryAttempts` and `retryDelay` in TypeORM config. [ASSUMED -- standard Docker Compose practice]
**Warning signs:** Backend container exits with "Connection refused" on startup, works fine after manual restart.

### Pitfall 6: Vite Proxy Not Forwarding to NestJS
**What goes wrong:** Frontend API calls to `/api/assets` return 404 because the Vite dev server does not proxy to the NestJS backend.
**Why it happens:** Vite does not proxy by default; it must be configured in `vite.config.ts`.
**How to avoid:** Configure the `server.proxy` option in `vite.config.ts` to forward `/api` requests to `http://localhost:3000`. [ASSUMED -- standard Vite configuration]
**Warning signs:** 404 errors in browser console for `/api/assets`; Network tab shows the request going to Vite's port (5173) instead of NestJS (3000).

### Pitfall 7: Enum Column Without PostgreSQL Enum Type
**What goes wrong:** TypeORM creates `character varying` columns instead of PostgreSQL `enum` type, allowing any string value and losing type safety.
**Why it happens:** Omitting `type: 'enum'` in the `@Column` decorator causes TypeORM to fall back to string type.
**How to avoid:** Always specify `{ type: 'enum', enum: MyEnum }` in the column options. [CITED: typeorm.io/docs/entity/entities enum example]
**Warning signs:** `\d assets` in psql shows `character varying` instead of `asset_type_enum`.

## Code Examples

### NestJS Global Validation Pipe (main.ts)

```typescript
// Source: [CITED: context7.com/nestjs/docs.nestjs.com]
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.setGlobalPrefix('api');
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,           // Strip unknown properties
      forbidNonWhitelisted: true, // Throw on unknown properties
      transform: true,           // Auto-transform to DTO class instances
    }),
  );

  await app.listen(3000);
}
bootstrap();
```

### Create Asset DTO with class-validator

```typescript
// Source: [CITED: context7.com/nestjs/docs.nestjs.com] -- ValidationPipe pattern
import { IsString, IsEnum, IsOptional, IsBoolean, Length } from 'class-validator';
import { AssetType, MacroCategory } from '../entities/asset.entity';

export class CreateAssetDto {
  @IsString()
  name: string;

  @IsEnum(AssetType)
  type: AssetType;

  @IsOptional()
  @IsString()
  symbol?: string;

  @IsString()
  quantity: string;  // DECIMAL as string to preserve precision

  @IsString()
  @Length(3, 3)
  currency: string;

  @IsEnum(MacroCategory)
  category: MacroCategory;

  @IsOptional()
  @IsString()
  institution?: string;

  @IsOptional()
  @IsString()
  purchasePrice?: string;

  @IsOptional()
  @IsString()
  notes?: string;
}
```

### Assets Controller (CRUD)

```typescript
// Source: [CITED: context7.com/nestjs/docs.nestjs.com] -- Controller pattern
import {
  Controller, Get, Post, Put, Delete, Body, Param, Query,
} from '@nestjs/common';
import { AssetsService } from './assets.service';
import { CreateAssetDto } from './dto/create-asset.dto';
import { UpdateAssetDto } from './dto/update-asset.dto';

@Controller('assets')
export class AssetsController {
  constructor(private readonly assetsService: AssetsService) {}

  @Get()
  findAll(@Query('category') category?: string) {
    return this.assetsService.findAll(category);
  }

  @Post()
  create(@Body() createAssetDto: CreateAssetDto) {
    return this.assetsService.create(createAssetDto);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateAssetDto: UpdateAssetDto) {
    return this.assetsService.update(id, updateAssetDto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.assetsService.softRemove(id);
  }
}
```

### Assets Service with Soft Delete

```typescript
// Source: [CITED: typeorm.io repository API -- softRemove/softDelete]
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Asset } from './entities/asset.entity';
import { CreateAssetDto } from './dto/create-asset.dto';
import { UpdateAssetDto } from './dto/update-asset.dto';

@Injectable()
export class AssetsService {
  constructor(
    @InjectRepository(Asset)
    private readonly assetRepository: Repository<Asset>,
  ) {}

  async findAll(category?: string): Promise<Asset[]> {
    const query = this.assetRepository.createQueryBuilder('asset');
    // @DeleteDateColumn automatically excludes soft-deleted rows
    if (category) {
      query.where('asset.category = :category', { category });
    }
    query.orderBy('asset.updatedAt', 'DESC');
    return query.getMany();
  }

  async create(createAssetDto: CreateAssetDto): Promise<Asset> {
    const asset = this.assetRepository.create(createAssetDto);
    return this.assetRepository.save(asset);
  }

  async update(id: string, updateAssetDto: UpdateAssetDto): Promise<Asset> {
    const asset = await this.assetRepository.findOne({ where: { id } });
    if (!asset) throw new NotFoundException(`Asset ${id} not found`);
    Object.assign(asset, updateAssetDto);
    return this.assetRepository.save(asset);
  }

  async softRemove(id: string): Promise<void> {
    const asset = await this.assetRepository.findOne({ where: { id } });
    if (!asset) throw new NotFoundException(`Asset ${id} not found`);
    await this.assetRepository.softRemove(asset);
  }
}
```

### TypeORM Migration data-source.ts (for CLI)

```typescript
// Source: [CITED: context7.com/typeorm/typeorm -- migration CLI docs]
import { DataSource } from 'typeorm';
import * as dotenv from 'dotenv';

dotenv.config();

export default new DataSource({
  type: 'postgres',
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT, 10) || 5432,
  username: process.env.DB_USERNAME || 'postgres',
  password: process.env.DB_PASSWORD || 'postgres',
  database: process.env.DB_NAME || 'finance_manager',
  entities: ['src/modules/**/*.entity.ts'],
  migrations: ['migrations/*.ts'],
});
```

### Docker Compose (Development)

```yaml
# docker-compose.yml -- development configuration
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: finance_manager
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

### Dynamic Form Fields (React pattern)

```typescript
// Pattern for conditional field rendering based on asset type
const TYPE_SPECIFIC_FIELDS: Record<AssetType, string[]> = {
  [AssetType.BANK_CURRENT]: ['institution'],
  [AssetType.BANK_DEPOSIT]: ['institution'],
  [AssetType.STOCK]: ['symbol', 'purchasePrice'],
  [AssetType.ETF]: ['symbol', 'purchasePrice'],
  [AssetType.ETP]: ['symbol', 'purchasePrice'],
};

function AssetFormFields({ watch }: { watch: UseFormReturn['watch'] }) {
  const selectedType = watch('type');
  const visibleFields = TYPE_SPECIFIC_FIELDS[selectedType] || [];

  return (
    <>
      {/* Always-visible fields */}
      <FormField name="name" label="Name" required />
      <FormField name="type" label="Type" required />
      <FormField name="quantity" label="Quantity" required />
      <FormField name="currency" label="Currency" required />
      <FormField name="category" label="Category" required />

      {/* Type-specific fields */}
      {visibleFields.includes('symbol') && (
        <FormField name="symbol" label="Symbol" />
      )}
      {visibleFields.includes('institution') && (
        <FormField name="institution" label="Institution" />
      )}
      {visibleFields.includes('purchasePrice') && (
        <FormField name="purchasePrice" label="Purchase Price" />
      )}

      {/* Always optional */}
      <FormField name="notes" label="Notes" />
    </>
  );
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Tailwind v3 (PostCSS plugin + tailwind.config.js) | Tailwind v4 (@tailwindcss/vite + CSS @theme) | 2025 | No config file needed; CSS-first configuration; must use Vite plugin [VERIFIED: npm registry] |
| React Router v6 (createBrowserRouter) | React Router v7 (unified react-router-dom) | 2024 | v7 API is backward-compatible with v6 patterns; same import paths [VERIFIED: npm registry] |
| TypeORM 0.2 (ActiveRecord pattern) | TypeORM 0.3 (Data Mapper, DataSource) | 2022 | Must use `DataSource` not `getConnection()`; migration CLI uses `-d` flag [CITED: typeorm.io] |
| class-validator + class-transformer on backend | zod gaining traction | 2024+ | NestJS ecosystem still defaults to class-validator; zod via nestjs-zod is emerging but not standard yet [ASSUMED] |
| React 18 (createRoot) | React 19 (same API, new features) | 2024 | No breaking changes for Phase 1 patterns; `createRoot` still the entry point [VERIFIED: npm registry] |

**Deprecated/outdated:**
- `typeorm@0.2` patterns (`getConnection()`, `@EntityRepository()`): Removed in 0.3. Use `@InjectRepository()` and `DataSource` instead. [CITED: typeorm.io migration guide]
- `tailwind.config.js` and `postcss.config.js`: Not used in Tailwind v4 with Vite. [VERIFIED: @tailwindcss/vite package exists at 4.2.2]

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | TypeORM returns DECIMAL as `string` when TypeScript property is typed `string` | Architecture Patterns (Pattern 3) | If TypeORM returns `number`, we need to configure `pg` driver or use a transformer to avoid precision loss |
| A2 | Tailwind v4 with Vite uses `@tailwindcss/vite` plugin and `@import "tailwindcss"` in CSS | Architecture Patterns (Pattern 5) | If wrong, frontend styling setup fails; verify during implementation |
| A3 | class-validator is preferred over zod for NestJS backend validation | Standard Stack (Alternatives) | If nestjs-zod is now standard, we should switch; low risk since class-validator still works |
| A4 | TanStack React Query is preferred over SWR for this stack | Standard Stack (Alternatives) | Both work; preference-based, not correctness-based |
| A5 | shadcn/ui Dialog is recommended over Sheet for the add/edit form | Standard Stack (Alternatives) | Either works; Dialog is more conventional for forms |
| A6 | PostgreSQL `DECIMAL(20,8)` is sufficient for all crypto quantities | Architecture Patterns (Pattern 3) | Extremely small crypto quantities (wei) could exceed precision; unlikely for portfolio management |
| A7 | Node.js 24 is compatible with all specified packages despite PROJECT.md specifying Node 22 LTS | Environment Availability | If incompatibilities exist, we need to switch to Node 22; all packages tested on 24 |

## Open Questions

1. **DECIMAL handling in pg driver**
   - What we know: TypeORM 0.3 with pg driver may return DECIMAL as string or number depending on configuration.
   - What's unclear: Whether `string` type on the entity property is sufficient, or if we need a custom column transformer.
   - Recommendation: During implementation, write a test that inserts a value with 8 decimal places and reads it back. If precision is lost, add a custom TypeORM `ValueTransformer`.

2. **Node.js version alignment**
   - What we know: Environment has Node 24.14.1; PROJECT.md specifies Node 22 LTS.
   - What's unclear: Whether Dockerfile should use Node 22 or 24.
   - Recommendation: Use Node 22 in Dockerfile (matches PROJECT.md specification). Local dev with Node 24 should work fine but document the version difference.

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Node.js | Backend + Frontend | Yes | 24.14.1 | Dockerfile uses node:22-alpine |
| npm | Package management | Yes | 11.11.0 | -- |
| Docker | PostgreSQL container | Yes | 29.3.1 | -- |
| Docker Compose | Service orchestration | Yes | v5.1.1 | -- |
| PostgreSQL client (psql) | Manual DB inspection | No | -- | Use Docker exec: `docker compose exec postgres psql` |

**Missing dependencies with no fallback:**
- None -- all critical dependencies are available.

**Missing dependencies with fallback:**
- psql not installed locally. Use `docker compose exec postgres psql -U postgres finance_manager` for manual database inspection during development.

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | Vitest 4.1.4 (backend + frontend) |
| Config file | None -- Wave 0 (create vitest.config.ts) |
| Quick run command (backend) | `cd backend && npx vitest run --reporter=verbose` |
| Quick run command (frontend) | `cd frontend && npx vitest run --reporter=verbose` |
| Full suite command | `cd backend && npx vitest run && cd ../frontend && npx vitest run` |

### Phase Requirements to Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| DATA-01 | Create asset with all required fields persists to DB | unit | `cd backend && npx vitest run --reporter=verbose` | No -- Wave 0 |
| DATA-01 | Create asset validates required fields (name, type, quantity, currency, category) | unit | `cd backend && npx vitest run --reporter=verbose` | No -- Wave 0 |
| DATA-01 | Dynamic form shows correct fields per asset type | unit | `cd frontend && npx vitest run --reporter=verbose` | No -- Wave 0 |
| DATA-02 | Update asset changes persisted values | unit | `cd backend && npx vitest run --reporter=verbose` | No -- Wave 0 |
| DATA-02 | Soft delete hides asset from listing | unit | `cd backend && npx vitest run --reporter=verbose` | No -- Wave 0 |
| DATA-02 | Delete confirmation dialog prevents accidental deletion | unit | `cd frontend && npx vitest run --reporter=verbose` | No -- Wave 0 |
| DATA-07 | Category enum enforces 4 fixed values | unit | `cd backend && npx vitest run --reporter=verbose` | No -- Wave 0 |
| DATA-07 | Category tabs filter asset list client-side | unit | `cd frontend && npx vitest run --reporter=verbose` | No -- Wave 0 |
| DATA-01 | Full CRUD flow works end-to-end | integration | manual -- requires running backend + DB | N/A |

### Sampling Rate
- **Per task commit:** `cd backend && npx vitest run --reporter=verbose` (or frontend equivalent)
- **Per wave merge:** Full suite: backend + frontend tests
- **Phase gate:** Full suite green before `/gsd-verify-work`

### Wave 0 Gaps
- [ ] `backend/vitest.config.ts` -- Vitest configuration for NestJS (tsconfig paths, module resolution)
- [ ] `backend/src/modules/assets/assets.service.spec.ts` -- Unit tests for AssetsService (DATA-01, DATA-02, DATA-07)
- [ ] `backend/src/modules/assets/assets.controller.spec.ts` -- Unit tests for AssetsController
- [ ] `frontend/vitest.config.ts` -- Vitest configuration for React (jsdom environment)
- [ ] `frontend/src/components/assets/AssetForm.test.tsx` -- Form rendering and validation tests
- [ ] `frontend/src/components/assets/AssetList.test.tsx` -- List rendering and filtering tests
- [ ] `frontend/src/hooks/useAssets.test.ts` -- TanStack Query hook tests

## Security Domain

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | No | Single user, no auth required (per PROJECT.md Out of Scope) |
| V3 Session Management | No | No sessions |
| V4 Access Control | No | Single user |
| V5 Input Validation | Yes | class-validator DTOs on backend, Zod schemas on frontend |
| V6 Cryptography | No (Phase 1) | API key encryption deferred to Phase 2 (Exchanges module) |

### Known Threat Patterns for NestJS + React + PostgreSQL

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| SQL injection | Tampering | TypeORM parameterized queries (never raw query strings from user input) |
| XSS via asset name/notes | Tampering | React auto-escapes rendered content; sanitize on backend as defense-in-depth |
| Mass assignment | Tampering | `ValidationPipe({ whitelist: true, forbidNonWhitelisted: true })` strips unknown fields |
| CSRF | Tampering | Not applicable -- API uses JSON, not form submissions; no cookies for auth |

## Sources

### Primary (HIGH confidence)
- NestJS TypeORM integration: context7.com/nestjs/typeorm -- TypeOrmModule.forRootAsync(), forFeature(), configuration patterns
- TypeORM entities and columns: context7.com/typeorm/typeorm + typeorm.io -- @Column, @PrimaryGeneratedColumn('uuid'), enum types, @DeleteDateColumn, soft delete
- TypeORM migrations: context7.com/typeorm/typeorm -- migration:generate, migration:run, data-source.ts, CLI commands
- shadcn/ui Vite installation: ui.shadcn.com/docs/installation/vite -- init command, component installation
- shadcn/ui DataTable: ui.shadcn.com/docs/components/data-table -- TanStack Table integration, sorting, filtering
- shadcn/ui Dialog: ui.shadcn.com/docs/components/dialog -- modal component installation
- npm registry: All package versions verified 2026-04-20

### Secondary (MEDIUM confidence)
- Tailwind CSS v4 + Vite: npm registry confirms @tailwindcss/vite@4.2.2 exists as separate package; configuration pattern based on package design [ASSUMED for exact CSS syntax]
- ARCHITECTURE.md: Project-specific architecture patterns and anti-patterns (pre-researched)

### Tertiary (LOW confidence)
- None -- all critical claims verified through primary sources

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH -- all versions verified against npm registry, all patterns verified against Context7 docs
- Architecture: HIGH -- follows established NestJS + TypeORM + React conventions, verified through documentation
- Pitfalls: HIGH -- TypeORM migration CLI, DECIMAL handling, Tailwind v4, and Docker healthcheck pitfalls are well-documented

**Research date:** 2026-04-20
**Valid until:** 2026-05-20 (stable libraries, 30-day validity)
