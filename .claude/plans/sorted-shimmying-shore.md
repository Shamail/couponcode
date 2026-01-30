# Couponcode Documentation Alignment Plan

## Overview

This plan aligns couponcode documentation with proven patterns from the **clubhouse-journal** project (`/Users/shamail/Development/clubhouse-journal`). The clubhouse-journal project serves as the reference implementation for all code examples and architectural decisions.

**Approach**: All code examples in couponcode docs should follow clubhouse-journal patterns. This plan provides high-level references—the executor should read the referenced files directly for implementation details.

---

## 1. Project Structure & Organization

### Reference: `/Users/shamail/Development/clubhouse-journal`

**Monorepo Architecture**:
- `apps/` - Application packages (mobile, api, admin)
- `packages/` - Shared libraries with `@clubhouse/*` namespace
- `infrastructure/` - SST infrastructure as code
- Feature-based organization across all layers

**Key Files**:
- Root structure: `package.json` (workspaces config)
- SST config: `sst.config.ts`
- TypeScript: `tsconfig.json` (path aliases with `@/`)

**Couponcode Docs to Update**:
- `docs/tech/TECH-00-architecture.md` - Align monorepo structure
- `docs/tech/TECH-07-deployment.md` - Reference SST patterns

---

## 2. Backend API Patterns

### Reference: `/Users/shamail/Development/clubhouse-journal/apps/api/src/`

**Framework**: Hono.js on AWS Lambda

**Architecture Layers**:
| Layer | Location | Purpose |
|-------|----------|---------|
| Routes | `routes/*.ts` | HTTP handlers, use router factory pattern |
| Services | `services/*.ts` | Business logic with dependency injection |
| Validators | `validators/*.ts` | Zod schemas for input validation |
| Middleware | `middleware/*.ts` | Auth, rate limiting, tracing, CORS |

**Key Patterns**:
- **Router Factory**: `(repositories: Repositories) => Hono` pattern
- **Service Factory**: `createXxxService(deps)` returns service object
- **Repository Pattern**: Typed data access via `@clubhouse/db`
- **Error Responses**: Consistent JSON format `{ error, details? }`

**Reference Files**:
- API factory: `/apps/api/src/index.ts`
- Route example: `/apps/api/src/routes/courses.ts`
- Service example: `/apps/api/src/services/courses.ts`
- Middleware: `/apps/api/src/middleware/admin.ts`, `rateLimit.ts`, `tracing.ts`

**Couponcode Docs to Update**:
- `docs/tech/TECH-06-api-contract.md` - Use Hono patterns
- `docs/tech/TECH-04-security.md` - Reference middleware stack

---

## 3. Database Patterns

### Reference: `/Users/shamail/Development/clubhouse-journal/packages/db/src/`

**ORM**: Drizzle ORM with PostgreSQL (Neon serverless)

**Structure**:
- `schema/*.ts` - Table definitions with Drizzle
- `repositories/*.ts` - Typed CRUD operations
- `migrations/*.ts` - Versioned SQL migrations
- `db.ts` - Database factory

**Key Patterns**:
- Schema uses `pgTable()` with typed columns
- Repositories export typed interfaces (`$inferSelect`, `$inferInsert`)
- Centralized `Repositories` interface combining all repositories
- Index definitions colocated with schema

**Reference Files**:
- Schema example: `/packages/db/src/schema/rounds.ts`
- Repository example: `/packages/db/src/repositories/courses.ts`
- Repository index: `/packages/db/src/repositories/index.ts`
- Drizzle config: `/packages/db/drizzle.config.ts`

**Couponcode Docs to Update**:
- `docs/tech/TECH-01-database.md` - Use Drizzle patterns
- `docs/tech/TECH-09-migrations.md` - Reference migration workflow
- `docs/data/DATA-schema.md` - Schema definition patterns

---

## 4. Mobile App Patterns (React Native + Expo)

### Reference: `/Users/shamail/Development/clubhouse-journal/apps/mobile/src/`

**Framework**: React Native 0.81 + Expo 54 with new architecture

**Project Structure**:
```
src/
├── app/           # Expo Router file-based routing
├── components/    # UI components (ui/ for primitives)
├── hooks/         # Custom hooks (65+ examples)
├── stores/        # Zustand state management
├── services/      # API & business logic
├── collections/   # TanStack DB local data
├── utils/         # Helpers
└── theme/         # Design tokens
```

**Key Patterns**:
- **Routing**: Expo Router with route groups `(tabs)/`, `(auth)/`
- **State**: Zustand stores with AsyncStorage persistence
- **Data Fetching**: TanStack Query with offline persistence
- **Styling**: NativeWind (Tailwind for RN) + Gluestack UI
- **Forms**: Gluestack FormControl components

**Reference Files**:
- Root layout: `/apps/mobile/src/app/_layout.tsx`
- Auth store: `/apps/mobile/src/stores/authStore.ts`
- API client: `/apps/mobile/src/services/api.ts`
- Query client: `/apps/mobile/src/services/queryClient.ts`
- Theme colors: `/apps/mobile/src/theme/colors.ts`

**Couponcode Docs to Update**:
- `docs/tech/TECH-05-mobile-stack.md` - Align with Expo patterns
- `docs/design/DES-04-design-tokens.md` - Use theme system

---

## 5. Data Fetching & Caching

### Reference: `/Users/shamail/Development/clubhouse-journal/apps/mobile/src/services/`

**Library**: TanStack Query v5

**Configuration**:
- Garbage collection: 1 day
- Stale times by entity type (courses: 1 day, rounds: 1 minute)
- Retry: 3 attempts with exponential backoff
- Persistence: AsyncStorage with `@tanstack/query-async-storage-persister`

**Offline Support**:
- Network detection via NetInfo
- Mutation queue with retry on reconnect
- Focus management via AppState

**Reference Files**:
- Query client setup: `/apps/mobile/src/services/queryClient.ts`
- Offline queue: `/apps/mobile/src/services/offlineQueue.ts`
- API wrapper: `/apps/mobile/src/services/api.ts`

**Couponcode Docs to Update**:
- `docs/design/DES-19-offline-patterns.md` - Reference offline queue
- `docs/tech/TECH-05-mobile-stack.md` - Query configuration

---

## 6. Authentication & Security

### Reference: `/Users/shamail/Development/clubhouse-journal/packages/auth/` and `/apps/api/src/middleware/`

**Provider**: AWS Cognito with passwordless OTP

**Patterns**:
- JWT verification via `aws-jwt-verify`
- RBAC with user roles (user, admin, super_admin)
- Rate limiting (standard: 100/min, sensitive: 50/hour)
- Distributed rate limiting via Upstash Redis

**Middleware Stack Order**:
1. CORS
2. Tracing (request IDs)
3. Authentication (JWT)
4. Rate Limiting
5. Authorization (role-based)

**Reference Files**:
- Auth middleware: `/packages/auth/src/middleware.ts`
- Admin middleware: `/apps/api/src/middleware/admin.ts`
- Rate limiting: `/apps/api/src/middleware/rateLimit.ts`
- Tracing: `/apps/api/src/middleware/tracing.ts`

**Couponcode Docs to Update**:
- `docs/tech/TECH-04-security.md` - Middleware patterns
- `docs/tech/TECH-10-security-compliance.md` - Auth implementation

---

## 7. UI Component Patterns

### Reference: `/Users/shamail/Development/clubhouse-journal/apps/mobile/src/components/`

**Library**: Gluestack UI v3 + NativeWind

**Component Organization**:
- `ui/` - Primitive components (Button, Card, Input, etc.)
- Feature components alongside features
- CVA (Class Variance Authority) for variants

**Theming**:
- CSS variables for colors (`--color-primary-500`)
- Light/dark mode via class-based switching
- Semantic color tokens (primary, secondary, error, success)

**Reference Files**:
- UI components: `/apps/mobile/src/components/ui/`
- Theme config: `/apps/mobile/tailwind.config.js`
- Color system: `/apps/mobile/src/theme/semantic.ts`

**Couponcode Docs to Update**:
- `docs/design/DES-10-components.md` - Component patterns
- `docs/design/DES-04-design-tokens.md` - Token system

---

## 8. Testing Patterns

### Reference: `/Users/shamail/Development/clubhouse-journal/apps/*/vitest.config.ts`

**Framework**: Vitest

**Patterns**:
- Tests colocated: `*.test.ts` next to source
- Hono route testing with `testClient`
- Factory functions for test dependencies
- Mock modules via `vi.mock()` and aliases

**Reference Files**:
- Mobile test config: `/apps/mobile/vitest.config.ts`
- Mobile mocks: `/apps/mobile/src/test/mocks/`
- API route tests: `/apps/api/src/routes/photobooks.test.ts`
- Service tests: `/apps/api/src/services/courses.test.ts`

**Couponcode Docs to Update**:
- Add testing section to technical docs

---

## 9. Configuration & Environment

### Reference: `/Users/shamail/Development/clubhouse-journal/`

**Environment Variables**:
- `.env.example` as template
- `EXPO_PUBLIC_*` prefix for mobile public vars
- SST secrets management for production

**Build Tools**:
- Mobile: Metro + Expo
- Admin: Vite
- API: Bun runtime

**Reference Files**:
- Env template: `.env.example`
- Mobile config: `/apps/mobile/app.config.js`
- Metro config: `/apps/mobile/metro.config.js`
- Tailwind: `/apps/mobile/tailwind.config.js`

**Couponcode Docs to Update**:
- `docs/tech/TECH-07-deployment.md` - Environment setup

---

## 10. Naming Conventions

### From clubhouse-journal

| Type | Convention | Example |
|------|------------|---------|
| Hooks | `use[Feature].ts` | `useRounds.ts` |
| Stores | `[feature]Store.ts` | `authStore.ts` |
| Services | `[feature].ts` | `courses.ts` |
| Routes | `[feature].ts` | `courses.ts` |
| Validators | `[feature].ts` | `courses.ts` |
| Components | `[Component].tsx` | `Card.tsx` |
| Tests | `[file].test.ts` | `courses.test.ts` |

**Import Patterns**:
- Monorepo: `@clubhouse/[package]`
- Local: `@/[path]` via path alias

---

## Implementation Priority

1. **Backend API** (TECH-06) - Hono + Drizzle patterns
2. **Mobile Stack** (TECH-05) - Expo Router + state management
3. **Database** (TECH-01) - Drizzle schema patterns
4. **Security** (TECH-04, TECH-10) - Auth middleware
5. **Design System** (DES-04, DES-10) - Component patterns
6. **Offline** (DES-19) - TanStack Query + offline queue

---

## Verification

When implementing:
1. Read the referenced clubhouse-journal files directly
2. Adapt patterns to couponcode's specific requirements
3. Ensure code examples follow the same architectural decisions
4. Run any example code to verify it works

---

## Summary

The clubhouse-journal project provides battle-tested patterns for:
- **Monorepo structure** with workspaces
- **API architecture** with Hono + Drizzle + Zod
- **Mobile architecture** with Expo + Zustand + TanStack Query
- **Offline-first** data management
- **Security** with AWS Cognito + RBAC
- **Testing** with Vitest + mocking strategies
- **Theming** with CSS variables + NativeWind

All couponcode documentation code examples should reference and follow these patterns.
