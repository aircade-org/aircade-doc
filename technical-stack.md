> [< back](./README.md)

# Technical Stack

This project uses a **Turborepo monorepo** to manage multiple applications and shared packages for a browser-based multiplayer gaming platform.

The stack combines Next.js 15 for server-side rendered frontends, NestJS for the real-time backend API, and Prisma with Supabase PostgreSQL for database management.

## Architecture Overview

The monorepo is organized into three main categories:

- **Apps**: Standalone deployable applications (Next.js frontend, NestJS backend)
- **Packages**: Shared libraries, configurations, and types used across apps
- **Database**: Prisma schema and migrations managed as a shared layer

All workspaces are managed with **pnpm workspaces** and orchestrated by **Turborepo** for optimized builds and caching.

## Monorepo Structure

```tkt
/
├── apps/
│   ├── web/              # Next.js 15 SSR frontend (console + controller UI)
│   └── api/              # NestJS backend (REST + WebSocket)
├── packages/
│   ├── database/         # Prisma schema, client, and migrations
│   ├── typescript-config/# Shared TypeScript configurations
│   ├── eslint-config/    # Shared ESLint rules
│   ├── ui/               # Shared React components (optional)
│   └── types/            # Shared TypeScript types and DTOs
├── turbo.json            # Turborepo pipeline configuration
├── package.json          # Root workspace configuration
└── pnpm-workspace.yaml   # pnpm workspace definition
```

## Technology Stack

### Frontend (apps/web)

- **Framework**: Next.js 15 with App Router for SSR and React Server Components
- **Language**: TypeScript 5.x
- **Styling**: Tailwind CSS (recommended) or CSS Modules
- **State Management**: React Context, Zustand, or React Query for server state
- **WebSocket Client**: Socket.IO client for real-time controller/console communication
- **Authentication**: NextAuth.js or custom session handling

**Key responsibilities**:

- Marketing pages (home, games catalog, info) with SEO-optimized SSR
- Console UI (`/play`) for big-screen game sessions
- Controller UI (`/join`) for mobile phone controls
- Consumes REST/GraphQL APIs and WebSocket namespaces from the backend

### Backend (apps/api)

- **Framework**: NestJS with TypeScript
- **Real-time**: Socket.IO with NestJS WebSocket Gateways for bidirectional communication
- **API Style**: REST or GraphQL (recommend REST for simplicity, GraphQL for complex queries)
- **Authentication**: JWT tokens with Passport.js strategies (local, OAuth)
- **Validation**: class-validator and class-transformer DTOs
- **ORM**: Prisma Client (imported from `@repo/database` package)

**Key responsibilities**:

- HTTP APIs for authentication, game catalog, user profiles, subscriptions
- WebSocket rooms for session lobbies and in-game real-time events
- Session code generation and validation
- Business logic orchestration and authorization

### Database (packages/database)

- **ORM**: Prisma 5.x
- **Database**: Supabase PostgreSQL (managed cloud Postgres)
- **Caching/Sessions**: Redis (recommended for ephemeral session codes and lobby state)
- **Migrations**: Prisma Migrate for version-controlled schema evolution

**Package structure**:

```txt
packages/database/
├── prisma/
│   ├── schema.prisma     # Data model definitions
│   └── migrations/       # Migration history
├── src/
│   └── index.ts          # Exports PrismaClient instance
├── package.json
└── tsconfig.json
```

**Key entities**:

- `User`: Authentication and profile data
- `Game`: Game metadata, player count, tags, free/premium flags
- `Session`: Console session with unique code, status, game reference
- `SessionPlayer`: Player slots with connection state and scores
- `Subscription`: Premium tier and payment tracking

**Setup with Supabase**:

1. Create a new Supabase project and copy the PostgreSQL connection string
2. Add to `.env`: `DATABASE_URL="postgresql://postgres:[password]@[host]:5432/postgres"`
3. Initialize Prisma: `npx prisma init --datasource-provider postgresql`
4. Define your schema in `prisma/schema.prisma` and run `npx prisma migrate dev`

## Shared Packages

### packages/typescript-config

Centralized TypeScript configuration extending across all apps and packages.

**Structure**:

```txt
packages/typescript-config/
├── base.json             # Base config for all workspaces
├── nextjs.json           # Next.js-specific overrides
├── nestjs.json           # NestJS-specific overrides
└── package.json
```

**Usage in apps**:

```json
{
  "extends": "@repo/typescript-config/nextjs.json",
  "compilerOptions": {
    "outDir": "dist"
  }
}
```

### packages/eslint-config

Shared ESLint rules for consistent code style.

**Includes**:

- Base rules for TypeScript projects
- Next.js-specific rules (extends `next/core-web-vitals`)
- NestJS backend rules
- Prettier integration for formatting

### packages/types

Shared TypeScript interfaces, DTOs, and type definitions used by both frontend and backend.

**Exports**:

- API request/response types
- WebSocket event payloads
- Shared domain models (UserDTO, GameDTO, SessionDTO, etc.)
- Ensures type safety across client-server boundaries

### packages/ui (optional)

Reusable React components shared across multiple frontends if you expand beyond the main web app.

## Turborepo Configuration

The `turbo.json` file defines the build pipeline and caching strategy.

**Example configuration**:

```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "db:migrate": {
      "cache": false
    },
    "db:generate": {
      "cache": false
    }
  }
}
```

**Key features**:

- Parallel task execution with dependency-aware ordering
- Remote caching for CI/CD speed improvements
- Incremental builds only rebuild changed packages

## Development Workflow

### Initial Setup

```bash
# Install dependencies (from root)
pnpm install

# Generate Prisma Client
pnpm --filter @repo/database db:generate

# Run migrations
pnpm --filter @repo/database db:migrate

# Start all apps in dev mode
pnpm dev
```

### Common Commands

```bash
# Build all apps and packages
pnpm build

# Run linting across workspace
pnpm lint

# Type-check all TypeScript
pnpm type-check

# Run tests (if configured)
pnpm test

# Start only the web app
pnpm --filter web dev

# Start only the API
pnpm --filter api dev
```

### Database Management

```bash
# Create a new migration
pnpm --filter @repo/database db:migrate

# Reset database (dev only)
pnpm --filter @repo/database db:reset

# Open Prisma Studio
pnpm --filter @repo/database db:studio

# Push schema without migration (prototyping)
pnpm --filter @repo/database db:push
```

## Environment Variables

Each app maintains its own `.env` file, with shared database connection managed in the `packages/database/.env`.

**apps/web/.env**:

```txt
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_WS_URL=ws://localhost:3001
NEXTAUTH_SECRET=your-secret
NEXTAUTH_URL=http://localhost:3000
```

**apps/api/.env**:

```txt
DATABASE_URL=postgresql://...
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-jwt-secret
CORS_ORIGIN=http://localhost:3000
```

**packages/database/.env**:

```txt
DATABASE_URL=postgresql://postgres:[password]@db.supabase.co:5432/postgres
```

## Deployment Strategy

- **Frontend**: Deploy `apps/web` to Vercel with automatic SSR support for Next.js
- **Backend**: Deploy `apps/api` to any Node.js hosting (Fly.io, Railway, AWS, DigitalOcean) with WebSocket support
- **Database**: Supabase PostgreSQL (managed, no separate deployment)
- **Redis**: Use Upstash Redis or managed Redis from cloud provider

**Build optimization**: Turborepo's remote caching can be configured with Vercel Remote Cache to speed up CI/CD pipelines across all apps.

> [< back](./README.md)
