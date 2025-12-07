---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments: ['docs/prd.md', 'docs/ux-design-specification.md']
workflowType: 'architecture'
lastStep: 8
status: 'complete'
completedAt: '2025-12-07'
project_name: 'StreamVibes'
user_name: 'Butters'
date: '2025-12-07'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
40 FRs spanning 8 domains - authentication (5), layout management (9), widgets (5), theming (5), overlay access (5), platform integration (5), dashboard (3), data isolation (3). The core architectural challenge is the overlay system which must accept real-time events from platform adapters and render them with minimal latency.

**Non-Functional Requirements:**
- Performance: <100ms overlay render, <50ms WebSocket latency, 2s dashboard load, 50ms DB queries
- Security: Encrypted tokens at rest, row-level isolation, webhook signature verification, HTTPS everywhere
- Reliability: Auto-reconnect WebSockets, graceful degradation, exponential backoff, 99.9% uptime target
- Scale: 1000 concurrent viewers per streamer

**Scale & Complexity:**
- Primary domain: Full-stack real-time web application
- Complexity level: Medium
- Estimated architectural components: 12-15 major components

### Technical Constraints & Dependencies

- Must support both Kick and Twitch from MVP launch (not phased)
- Platform API limitations dictate adapter patterns (different auth flows, different event systems)
- OBS browser source compatibility requires fixed 1920x1080 overlay rendering
- Streamers bring their own API keys for optional AI features (post-MVP)

### Cross-Cutting Concerns Identified

1. **Tenant Isolation** - Every data access must enforce `streamer_id` context
2. **WebSocket Lifecycle** - Connection management, reconnection, message routing
3. **OAuth Token Management** - Storage, refresh, per-platform handling
4. **Platform Rate Limiting** - API call budgeting across all features
5. **Real-Time Event Flow** - Platform → Backend → Overlay pipeline

## Starter Template & Infrastructure

### Primary Technology Domain

Full-stack real-time web application based on project requirements analysis. Key drivers:
- Multi-tenant SaaS with real-time WebSocket requirements
- Dual-platform integration (Kick + Twitch)
- Drag-and-drop UI with live preview
- Growth-oriented architecture

### Infrastructure Decision: Self-Hosted with Coolify

**Rationale for Self-Hosting:**
- WebSocket connections require persistent connections (Vercel has 30s-5min timeouts)
- Predictable costs at scale (1000 concurrent viewers per streamer)
- Full control over infrastructure for real-time features
- No vendor lock-in

**Platform: [Coolify](https://github.com/coollabsio/coolify)**
- Open-source, self-hostable PaaS (Vercel/Heroku alternative)
- Git-push deployments with automatic Docker builds
- Built-in PostgreSQL, Redis provisioning
- Automatic SSL via Let's Encrypt
- Beautiful UI for service management

### Monorepo Structure: Bun Workspace

**Initialization:**
```bash
bun init
```

**Workspace Structure:**
```
streaming-system/
├── package.json          # Bun workspace root
├── bun.lockb
├── apps/
│   ├── web/              # Next.js frontend + admin dashboard
│   └── ws-server/        # WebSocket server (Elysia or Hono)
├── packages/
│   ├── shared/           # Shared types, utils, constants
│   ├── platform-adapters/# Kick + Twitch API adapters
│   └── db/               # Database schema (Drizzle)
```

**Root package.json:**
```json
{
  "name": "streamvibes",
  "workspaces": ["apps/*", "packages/*"]
}
```

### Architectural Decisions Provided by Stack

**Runtime & Package Manager:**
- Bun for fast installs, native TypeScript, built-in test runner

**Frontend Framework:**
- Next.js 15 with App Router
- `output: 'standalone'` for Docker deployment
- shadcn/ui + Tailwind CSS (per UX spec)

**Backend/WebSocket:**
- Separate WebSocket server (Elysia or Hono on Bun)
- Full control over persistent connections
- No serverless timeout limitations

**Database:**
- PostgreSQL (via Coolify one-click)
- Drizzle ORM for type-safe queries

**Caching/Pub-Sub:**
- Redis (via Coolify one-click)
- Used for WebSocket message fanout and session caching

**Deployment:**
- Docker containers managed by Coolify
- Traefik for routing and automatic HTTPS

### What This Stack Provides vs. Vercel

| Feature | This Stack | Vercel |
|---------|-----------|--------|
| SSR/SSG | ✅ | ✅ |
| API Routes | ✅ | ✅ |
| WebSockets | ✅ Full control | ⚠️ Limited timeouts |
| ISR | ✅ (needs shared cache for horizontal scale) | ✅ Easy |
| Edge Functions | ❌ Not needed | ✅ |
| Image Optimization | ✅ Self-serve or Cloudflare | ✅ Built-in CDN |
| Automatic HTTPS | ✅ Via Traefik | ✅ |
| Cost Control | ✅ Fixed server cost | ⚠️ Usage-based |

**Note:** Project initialization and Coolify setup should be the first implementation story

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Authentication: Better Auth + Arctic for OAuth
- Database: PostgreSQL 16 with Drizzle ORM
- Real-time: Elysia WebSocket server on Bun
- Deployment: Docker on Coolify

**Important Decisions (Shape Architecture):**
- State management: TanStack Query v5
- Form handling: React Hook Form + Zod
- Drag-and-drop: @dnd-kit

**Deferred Decisions (Post-MVP):**
- CDN configuration (Cloudflare if needed)
- Horizontal scaling strategy
- Analytics/monitoring stack expansion

### Data Architecture

**ORM: Drizzle v0.45.x**
- SQL-like type safety, lightweight (~7kb)
- Native PostgreSQL identity columns (2025 best practice)
- Drizzle Kit for SQL-based migrations

**Validation: Zod v3.x**
- Shared schemas between frontend/backend
- React Hook Form integration
- Runtime type checking

**Database: PostgreSQL 16**
- Row-level security capable for tenant isolation
- JSONB for flexible widget configurations
- Identity columns over serial (modern Postgres)

### Authentication & Security

**Framework: [Better Auth](https://www.better-auth.com/)**
- Modern Lucia replacement (Lucia deprecated March 2025)
- Native Drizzle adapter
- Plugin ecosystem for future 2FA, orgs

**OAuth: [Arctic](https://arcticjs.dev/)**
- Built-in [Kick provider](https://arcticjs.dev/providers/kick)
- Built-in [Twitch provider](https://arcticjs.dev/providers/twitch)
- Lightweight, Fetch-based, maintained by Lucia creator

**Session Strategy: Database Sessions**
- Stored in PostgreSQL
- Enables tenant-scoped session queries
- Better revocation than JWT

**Security Middleware:**
- CSRF protection (Better Auth built-in)
- Rate limiting on auth endpoints
- Webhook signature verification (platform-specific)

### API & Communication Patterns

**Dashboard API: Next.js API Routes (REST)**
- `/api/layouts/*` - CRUD for overlay layouts
- `/api/widgets/*` - Widget configuration
- `/api/themes/*` - Theme management
- `/api/settings/*` - Account settings

**Real-time Server: [Elysia](https://elysiajs.com/) v1.2 on Bun**
- Separate service in monorepo (`apps/ws-server`)
- Native WebSocket (no Socket.io overhead)
- Redis pub/sub for message fanout

**Event Flow:**
```
Platform (Kick/Twitch) → Platform Adapter → Redis Pub/Sub → Elysia WS → Overlay Client
```

### Frontend Architecture

**State Management: TanStack Query v5**
- Server state caching and synchronization
- Optimistic updates for dashboard UX
- No global state library needed (React context for theme)

**Forms: React Hook Form + Zod**
- Type-safe form validation
- Shared schemas with backend

**Drag-and-Drop: @dnd-kit**
- Accessible, keyboard-friendly
- Works with React 18 concurrent features
- Used for layout builder

**Component Architecture:**
- shadcn/ui base components
- Compound component pattern for widgets
- CSS variables for dynamic theming

### Infrastructure & Deployment

**Container Strategy:**
- `apps/web` → Next.js standalone Docker image
- `apps/ws-server` → Elysia Docker image
- Both deployed via Coolify

**Service Dependencies (Coolify one-click):**
- PostgreSQL 16
- Redis 7

**Environment Configuration:**
- `.env.local` for development
- Coolify environment variables for production
- Secrets encrypted at rest

### Decision Impact Analysis

**Implementation Sequence:**
1. Bun workspace + Coolify setup
2. Database schema (Drizzle)
3. Better Auth + Arctic OAuth
4. Next.js dashboard shell
5. Elysia WebSocket server
6. Platform adapters (Kick, Twitch)
7. Layout builder + widgets
8. Public overlay pages

**Cross-Component Dependencies:**
- Auth affects: API routes, WS connections, tenant isolation
- Drizzle schema affects: All data access, migrations
- Redis affects: WS fanout, session caching
- Platform adapters affect: Event flow, OAuth tokens

## Implementation Patterns & Consistency Rules

### Naming Conventions

**Database (Drizzle/PostgreSQL):**
| Element | Convention | Example |
|---------|------------|---------|
| Tables | snake_case, plural | `streamers`, `overlay_layouts`, `widget_configs` |
| Columns | snake_case | `streamer_id`, `created_at`, `is_active` |
| Foreign keys | `{table}_id` | `streamer_id`, `layout_id` |
| Indexes | `idx_{table}_{columns}` | `idx_streamers_email` |
| Primary keys | `id` (identity column) | `id` |

**API Endpoints (Next.js Routes):**
| Element | Convention | Example |
|---------|------------|---------|
| Routes | kebab-case, plural | `/api/layouts`, `/api/widget-configs` |
| Parameters | camelCase | `/api/layouts/[layoutId]` |
| Query params | camelCase | `?streamerId=123&isActive=true` |

**TypeScript/React:**
| Element | Convention | Example |
|---------|------------|---------|
| Components | PascalCase file + export | `LayoutEditor.tsx` |
| Hooks | camelCase with `use` prefix | `useLayouts.ts` |
| Utilities | camelCase | `formatDate.ts` |
| Types/Interfaces | PascalCase | `Streamer`, `LayoutConfig` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_WIDGETS`, `WS_RECONNECT_DELAY` |
| Zod schemas | camelCase with `Schema` suffix | `layoutSchema`, `widgetConfigSchema` |

### Project Structure Patterns

```
streaming-system/
├── apps/
│   ├── web/                       # Next.js app
│   │   ├── app/                   # App Router
│   │   │   ├── (auth)/            # Auth-required routes (dashboard)
│   │   │   ├── (public)/          # Public routes (overlays)
│   │   │   └── api/               # API routes
│   │   ├── components/
│   │   │   ├── ui/                # shadcn/ui components
│   │   │   ├── layout/            # Layout-specific components
│   │   │   └── widgets/           # Widget components
│   │   ├── hooks/                 # Custom React hooks
│   │   ├── lib/                   # Utilities, clients
│   │   └── __tests__/             # Co-located tests
│   └── ws-server/                 # Elysia WebSocket server
│       ├── src/
│       │   ├── handlers/          # WS message handlers
│       │   ├── services/          # Business logic
│       │   └── index.ts
│       └── __tests__/
├── packages/
│   ├── db/                        # Drizzle schema
│   │   ├── schema/                # Table definitions
│   │   ├── migrations/            # SQL migrations
│   │   └── index.ts
│   ├── shared/                    # Shared code
│   │   ├── types/                 # TypeScript types
│   │   ├── constants/             # Shared constants
│   │   ├── schemas/               # Zod validation schemas
│   │   └── utils/
│   └── platform-adapters/         # Kick + Twitch
│       ├── kick/
│       ├── twitch/
│       └── types.ts               # Adapter interface
```

**Test Location:** Co-located `__tests__` folders mirroring source structure.

### API Response Patterns

**Success Response:**
```typescript
// Single item
{ data: T }

// Collection with pagination
{ data: T[], meta: { page: number, total: number, hasMore: boolean } }
```

**Error Response:**
```typescript
{
  error: {
    code: string,      // "VALIDATION_ERROR" | "NOT_FOUND" | "UNAUTHORIZED" | "FORBIDDEN"
    message: string,   // Human-readable message
    details?: unknown  // Validation errors array, etc.
  }
}
```

**HTTP Status Codes:**
| Code | Usage |
|------|-------|
| 200 | Success (GET, PUT, PATCH) |
| 201 | Created (POST) |
| 204 | No Content (DELETE) |
| 400 | Validation Error |
| 401 | Not Authenticated |
| 403 | Forbidden (tenant mismatch) |
| 404 | Not Found |
| 500 | Internal Server Error |

### WebSocket Message Patterns

**Message Format:**
```typescript
{
  type: string,         // "{domain}:{action}" format
  payload: T,
  timestamp: string,    // ISO 8601
  streamerId: string    // Tenant context
}
```

**Event Types:**
- `chat:message`, `chat:clear`
- `event:follow`, `event:subscribe`, `event:tip`
- `overlay:update`, `overlay:refresh`
- `widget:config-change`

### Data Format Standards

| Data Type | Format | Example |
|-----------|--------|---------|
| Dates (API/WS) | ISO 8601 string | `"2025-12-07T15:30:00Z"` |
| Dates (DB) | `timestamp with time zone` | PostgreSQL native |
| IDs | String | UUID or nanoid |
| Booleans | `true`/`false` | Never `1`/`0` |
| JSON fields | camelCase keys | `{ "widgetType": "chat" }` |
| Null values | Explicit `null` | Never `undefined` in responses |

### Tenant Isolation Pattern (CRITICAL)

**Every database query MUST include tenant context:**
```typescript
// ✅ CORRECT - Always filter by streamerId
const layouts = await db.query.layouts.findMany({
  where: eq(layouts.streamerId, session.streamerId)
});

// ❌ WRONG - Never query without tenant filter
const layouts = await db.query.layouts.findMany();
```

**API routes MUST verify ownership:**
```typescript
const layout = await getLayout(layoutId);
if (layout.streamerId !== session.streamerId) {
  return Response.json(
    { error: { code: "FORBIDDEN", message: "Access denied" } },
    { status: 403 }
  );
}
```

### Error Handling Patterns

**API Routes:**
```typescript
try {
  // business logic
} catch (error) {
  if (error instanceof ValidationError) {
    return Response.json(
      { error: { code: "VALIDATION_ERROR", message: error.message, details: error.errors } },
      { status: 400 }
    );
  }
  console.error("[API] Unhandled:", error);
  return Response.json(
    { error: { code: "INTERNAL_ERROR", message: "Something went wrong" } },
    { status: 500 }
  );
}
```

**Frontend:**
- React Error Boundaries for component crashes
- TanStack Query `onError` for API errors
- Toast notifications for user-facing errors
- Never expose stack traces to users

### Enforcement Rules

**All AI Agents MUST:**
1. Include `streamerId` in every database query
2. Follow exact naming conventions (no exceptions)
3. Return API responses in standard format
4. Use ISO 8601 for all date strings
5. Co-locate tests in `__tests__` folders
6. Import shared types from `@streamvibes/shared`
7. Never query data without tenant context verification

## Project Structure & Boundaries

### Complete Project Directory Structure

```
streaming-system/
├── README.md
├── package.json                    # Bun workspace root
├── bun.lockb
├── .gitignore
├── .env.example
├── docker-compose.yml              # Local dev: Postgres + Redis
├── docker-compose.prod.yml         # Production reference
├── .github/
│   └── workflows/
│       ├── ci.yml                  # Lint, test, typecheck
│       └── deploy.yml              # Coolify webhook trigger
│
├── apps/
│   ├── web/                        # Next.js 15 App
│   │   ├── package.json
│   │   ├── next.config.ts
│   │   ├── tailwind.config.ts
│   │   ├── tsconfig.json
│   │   ├── Dockerfile
│   │   ├── .env.local
│   │   ├── app/
│   │   │   ├── layout.tsx
│   │   │   ├── globals.css
│   │   │   ├── (public)/           # Public routes
│   │   │   │   ├── page.tsx        # Landing
│   │   │   │   ├── [platform]/
│   │   │   │   │   └── [channel]/
│   │   │   │   │       ├── overlay/page.tsx
│   │   │   │   │       ├── chat/page.tsx
│   │   │   │   │       ├── events/page.tsx
│   │   │   │   │       └── goals/page.tsx
│   │   │   │   └── auth/
│   │   │   │       ├── login/page.tsx
│   │   │   │       ├── callback/[provider]/route.ts
│   │   │   │       └── logout/route.ts
│   │   │   ├── (dashboard)/        # Auth-required
│   │   │   │   ├── layout.tsx
│   │   │   │   ├── dashboard/page.tsx
│   │   │   │   ├── overlay/page.tsx
│   │   │   │   ├── theme/page.tsx
│   │   │   │   ├── widgets/page.tsx
│   │   │   │   └── settings/page.tsx
│   │   │   └── api/
│   │   │       ├── auth/[...betterauth]/route.ts
│   │   │       ├── layouts/
│   │   │       │   ├── route.ts
│   │   │       │   └── [layoutId]/route.ts
│   │   │       ├── widgets/route.ts
│   │   │       ├── themes/route.ts
│   │   │       ├── settings/route.ts
│   │   │       └── webhooks/
│   │   │           ├── kick/route.ts
│   │   │           └── twitch/route.ts
│   │   ├── components/
│   │   │   ├── ui/                 # shadcn/ui
│   │   │   ├── layout/             # Dashboard layout
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   ├── Header.tsx
│   │   │   │   └── PlatformSwitcher.tsx
│   │   │   ├── editor/             # Layout builder
│   │   │   │   ├── LayoutEditor.tsx
│   │   │   │   ├── GridCanvas.tsx
│   │   │   │   ├── WidgetPicker.tsx
│   │   │   │   └── SettingsPanel.tsx
│   │   │   ├── widgets/            # Widget components
│   │   │   │   ├── ChatWidget.tsx
│   │   │   │   ├── EventsWidget.tsx
│   │   │   │   ├── GoalsWidget.tsx
│   │   │   │   └── TipsWidget.tsx
│   │   │   ├── overlay/            # OBS overlay
│   │   │   │   ├── OverlayContainer.tsx
│   │   │   │   └── OverlayWidget.tsx
│   │   │   └── auth/
│   │   │       ├── LoginButton.tsx
│   │   │       └── UserMenu.tsx
│   │   ├── hooks/
│   │   │   ├── useLayouts.ts
│   │   │   ├── useWidgets.ts
│   │   │   ├── useTheme.ts
│   │   │   ├── useWebSocket.ts
│   │   │   └── useSession.ts
│   │   ├── lib/
│   │   │   ├── auth.ts
│   │   │   ├── api-client.ts
│   │   │   ├── query-client.ts
│   │   │   └── utils.ts
│   │   ├── providers/
│   │   │   ├── QueryProvider.tsx
│   │   │   └── ThemeProvider.tsx
│   │   └── __tests__/
│   │
│   └── ws-server/                  # Elysia WebSocket
│       ├── package.json
│       ├── tsconfig.json
│       ├── Dockerfile
│       ├── src/
│       │   ├── index.ts
│       │   ├── config.ts
│       │   ├── handlers/
│       │   │   ├── overlay.ts
│       │   │   ├── chat.ts
│       │   │   └── events.ts
│       │   ├── services/
│       │   │   ├── redis.ts
│       │   │   ├── connection-manager.ts
│       │   │   └── message-router.ts
│       │   └── types.ts
│       └── __tests__/
│
├── packages/
│   ├── db/                         # Drizzle Database
│   │   ├── package.json
│   │   ├── drizzle.config.ts
│   │   ├── index.ts
│   │   ├── schema/
│   │   │   ├── index.ts
│   │   │   ├── streamers.ts
│   │   │   ├── platform-connections.ts
│   │   │   ├── layouts.ts
│   │   │   ├── widgets.ts
│   │   │   ├── themes.ts
│   │   │   └── sessions.ts
│   │   └── migrations/
│   │
│   ├── shared/                     # Shared Types & Utils
│   │   ├── package.json
│   │   ├── index.ts
│   │   ├── types/
│   │   │   ├── index.ts
│   │   │   ├── streamer.ts
│   │   │   ├── layout.ts
│   │   │   ├── widget.ts
│   │   │   ├── theme.ts
│   │   │   ├── events.ts
│   │   │   └── websocket.ts
│   │   ├── schemas/
│   │   │   ├── index.ts
│   │   │   ├── layoutSchema.ts
│   │   │   ├── widgetSchema.ts
│   │   │   └── themeSchema.ts
│   │   ├── constants/
│   │   │   ├── index.ts
│   │   │   ├── widgetTypes.ts
│   │   │   └── platforms.ts
│   │   └── utils/
│   │
│   └── platform-adapters/          # Kick + Twitch
│       ├── package.json
│       ├── index.ts
│       ├── types.ts
│       ├── kick/
│       │   ├── index.ts
│       │   ├── auth.ts
│       │   ├── chat.ts
│       │   ├── webhooks.ts
│       │   └── api.ts
│       └── twitch/
│           ├── index.ts
│           ├── auth.ts
│           ├── chat.ts
│           ├── eventsub.ts
│           └── api.ts
```

### Architectural Boundaries

**Service Communication Flow:**
```
┌─────────────────────────────────────────────────────────────┐
│                      EXTERNAL                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │   Kick   │  │  Twitch  │  │   OBS    │  │ Browser  │    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘    │
└───────┼─────────────┼─────────────┼─────────────┼──────────┘
        │             │             │             │
        ▼             ▼             │             │
┌───────────────────────────┐      │             │
│   platform-adapters       │      │             │
└───────────┬───────────────┘      │             │
            │                      │             │
            ▼                      │             │
┌───────────────────────────┐      │             │
│      Redis Pub/Sub        │◄─────┼─────────────┤
└───────────┬───────────────┘      │             │
            │                      │             │
            ▼                      ▼             │
┌───────────────────────────┐ ┌─────────────────┴────────────┐
│      ws-server            │ │         apps/web              │
│      (Elysia)             │ │       (Next.js)               │
└───────────────────────────┘ └───────────────────────────────┘
            │                              │
            └──────────────┬───────────────┘
                           ▼
                ┌─────────────────────┐
                │    PostgreSQL       │
                └─────────────────────┘
```

### Requirements to Structure Mapping

| FR Category | Primary Location | Database Schema |
|-------------|------------------|-----------------|
| Auth (FR1-5) | `app/(public)/auth/`, `api/auth/` | `sessions.ts` |
| Layouts (FR6-14) | `app/(dashboard)/overlay/`, `api/layouts/` | `layouts.ts` |
| Widgets (FR15-19) | `components/widgets/*`, `api/widgets/` | `widgets.ts` |
| Themes (FR20-24) | `app/(dashboard)/theme/`, `api/themes/` | `themes.ts` |
| Overlay (FR25-29) | `app/(public)/[platform]/[channel]/*` | - |
| Platform (FR30-34) | `packages/platform-adapters/*` | `platform-connections.ts` |
| Dashboard (FR35-37) | `app/(dashboard)/*` | - |
| Isolation (FR38-40) | All schemas include `streamer_id` | Row-level filtering |

### Integration Points

**Internal Communication:**
| From | To | Method |
|------|----|--------|
| Dashboard → API | REST | TanStack Query |
| API → Database | Drizzle | Direct queries |
| Platform Adapters → Redis | Pub/Sub | ioredis |
| Redis → WS Server | Pub/Sub | ioredis subscriber |
| WS Server → Overlay | WebSocket | Native WS |

**External Integrations:**
| Service | Package Location | Library |
|---------|------------------|---------|
| Kick OAuth | `platform-adapters/kick/auth.ts` | Arctic |
| Kick Chat | `platform-adapters/kick/chat.ts` | WebSocket |
| Twitch OAuth | `platform-adapters/twitch/auth.ts` | Arctic |
| Twitch Chat | `platform-adapters/twitch/chat.ts` | IRC pattern |
| Twitch EventSub | `platform-adapters/twitch/eventsub.ts` | Webhook |

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:**
All technology choices verified compatible:
- Bun ↔ Next.js 15, Drizzle, Elysia (all Bun-native or compatible)
- Drizzle ↔ PostgreSQL 16, Better Auth (native adapters)
- Better Auth ↔ Arctic, Next.js (designed for this stack)
- Elysia 1.2 ↔ Bun, Redis (native WebSocket support)

**Pattern Consistency:**
- Naming conventions align with TypeScript/React ecosystem standards
- Database patterns match Drizzle idioms (snake_case tables, identity columns)
- API patterns follow Next.js App Router conventions
- WebSocket patterns use Elysia native approach

**Structure Alignment:**
- Bun workspace structure supports all packages with `workspace:*` protocol
- Next.js App Router structure follows official conventions
- Clear separation of `apps/` and `packages/` enables clean dependency boundaries

### Requirements Coverage Validation ✅

**Functional Requirements Coverage (40 FRs):**

| Category | Count | Architectural Support |
|----------|-------|----------------------|
| Auth (FR1-5) | 5 | Better Auth + Arctic + sessions schema |
| Layouts (FR6-14) | 9 | layouts schema + API routes + editor components |
| Widgets (FR15-19) | 5 | widgets schema + component library + Zod schemas |
| Themes (FR20-24) | 5 | themes schema + CSS variables + ThemeProvider |
| Overlay (FR25-29) | 5 | Public routes + WS server + Redis pub/sub |
| Platform (FR30-34) | 5 | platform-adapters package + webhook handlers |
| Dashboard (FR35-37) | 3 | (dashboard) route group + layout components |
| Isolation (FR38-40) | 3 | streamer_id on all tables + enforcement rules |

**Non-Functional Requirements Coverage (20 NFRs):**

| Category | Architectural Support |
|----------|----------------------|
| Performance (NFR1-5) | Redis caching, Elysia WS (<50ms), Drizzle queries |
| Security (NFR6-12) | Better Auth encryption, row-level isolation, HTTPS |
| Reliability (NFR13-17) | WS auto-reconnect, graceful degradation patterns |
| Integration (NFR18-20) | Platform adapters, rate limiting, token refresh |

### Implementation Readiness Validation ✅

**Decision Completeness:**
- All critical decisions documented with specific versions
- Technology rationale provided for each choice
- Integration patterns specified with code examples

**Structure Completeness:**
- ~100 files/directories explicitly defined
- All routes mapped to functional requirements
- Package boundaries clearly established

**Pattern Completeness:**
- Naming conventions cover DB, API, and code
- API response/error formats fully specified
- WebSocket message formats defined
- Tenant isolation rules include code examples

### Gap Analysis Results

**Critical Gaps:** None identified

**Post-MVP Enhancement Areas:**
1. Monitoring/observability stack (use Coolify built-in initially)
2. CDN configuration (add Cloudflare when traffic demands)
3. Horizontal scaling patterns (single instance sufficient for MVP)

### Architecture Completeness Checklist

**✅ Requirements Analysis**
- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed (Medium)
- [x] Technical constraints identified
- [x] Cross-cutting concerns mapped (5 concerns)

**✅ Architectural Decisions**
- [x] Critical decisions documented with versions
- [x] Technology stack fully specified
- [x] Integration patterns defined
- [x] Performance considerations addressed

**✅ Implementation Patterns**
- [x] Naming conventions established
- [x] Structure patterns defined
- [x] Communication patterns specified
- [x] Process patterns documented

**✅ Project Structure**
- [x] Complete directory structure defined
- [x] Component boundaries established
- [x] Integration points mapped
- [x] Requirements to structure mapping complete

### Architecture Readiness Assessment

**Overall Status:** ✅ READY FOR IMPLEMENTATION

**Confidence Level:** HIGH

**Key Strengths:**
1. Modern, well-maintained stack (Bun, Drizzle, Better Auth, Elysia)
2. Clear tenant isolation patterns prevent data leaks
3. WebSocket architecture provides full control for real-time features
4. Dual-platform support (Kick + Twitch) designed from day one

**Areas for Future Enhancement:**
- Add observability stack post-MVP
- Configure CDN when traffic scales
- Document horizontal scaling when needed

### Implementation Handoff

**AI Agent Guidelines:**
1. Follow all architectural decisions exactly as documented
2. Use implementation patterns consistently across all components
3. Respect project structure and boundaries
4. Include `streamerId` in EVERY database query
5. Refer to this document for all architectural questions

**First Implementation Steps:**
```bash
# 1. Initialize Bun workspace
bun init
# Configure workspaces in package.json

# 2. Set up local development
docker-compose up -d  # Postgres + Redis

# 3. Create packages/db
# - Drizzle schema with all tables
# - Initial migration

# 4. Set up apps/web
# - Next.js 15 with App Router
# - Better Auth configuration
# - Arctic OAuth providers

# 5. Create apps/ws-server
# - Elysia WebSocket server
# - Redis pub/sub integration
```

## Architecture Completion Summary

### Workflow Completion

**Architecture Decision Workflow:** COMPLETED ✅
**Total Steps Completed:** 8
**Date Completed:** 2025-12-07
**Document Location:** docs/architecture.md

### Final Architecture Deliverables

**Complete Architecture Document**
- All architectural decisions documented with specific versions
- Implementation patterns ensuring AI agent consistency
- Complete project structure with all files and directories
- Requirements to architecture mapping
- Validation confirming coherence and completeness

**Implementation Ready Foundation**
- 15+ architectural decisions made
- 7 implementation pattern categories defined
- 5 major architectural components specified
- 40 functional + 20 non-functional requirements fully supported

**AI Agent Implementation Guide**
- Technology stack with verified versions
- Consistency rules that prevent implementation conflicts
- Project structure with clear boundaries
- Integration patterns and communication standards

### Quality Assurance Checklist

**✅ Architecture Coherence**
- [x] All decisions work together without conflicts
- [x] Technology choices are compatible
- [x] Patterns support the architectural decisions
- [x] Structure aligns with all choices

**✅ Requirements Coverage**
- [x] All functional requirements are supported
- [x] All non-functional requirements are addressed
- [x] Cross-cutting concerns are handled
- [x] Integration points are defined

**✅ Implementation Readiness**
- [x] Decisions are specific and actionable
- [x] Patterns prevent agent conflicts
- [x] Structure is complete and unambiguous
- [x] Examples are provided for clarity

---

**Architecture Status:** READY FOR IMPLEMENTATION ✅

**Next Phase:** Begin implementation using the architectural decisions and patterns documented herein.

**Document Maintenance:** Update this architecture when major technical decisions are made during implementation.
