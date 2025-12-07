---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments: ['docs/prd.md', 'docs/architecture.md', 'docs/ux-design-specification.md']
workflowType: 'epics-and-stories'
lastStep: 8
project_name: 'StreamVibes'
user_name: 'Butters'
date: '2025-12-07'
status: 'complete'
---

# StreamVibes - Epic Breakdown

**Author:** Butters
**Date:** 2025-12-07
**Project Level:** SaaS Platform
**Target Scale:** Multi-tenant streaming overlay platform
**Status:** Complete

---

## Overview

This document provides the complete epic and story breakdown for StreamVibes, decomposing the requirements from the [PRD](./prd.md) into implementable stories.

**References:**
- [PRD](./prd.md) - Product Requirements Document
- [Architecture](./architecture.md) - Technical Architecture
- [UX Design](./ux-design-specification.md) - UX Design Specification

### Epic Summary

| Epic | Title | Stories | FRs Covered |
|------|-------|---------|-------------|
| 1 | Foundation & Project Setup | 6 | FR38-40 |
| 2 | Streamer Authentication | 6 | FR1-5, FR35 |
| 3 | Dashboard & Navigation | 4 | FR35-37 |
| 4 | Layout Builder Core | 8 | FR6-14 |
| 5 | Widget System | 6 | FR15-19 |
| 6 | Theme Customization | 6 | FR20-24 |
| 7 | Platform Adapters | 6 | FR30-34 |
| 8 | Live Overlay Rendering | 5 | FR25-29 |

**Total: 8 Epics, 47 Stories covering all 40 Functional Requirements**

---

## Functional Requirements Inventory

| ID | Category | Description |
|----|----------|-------------|
| **Authentication & Account Management** |||
| FR1 | Auth | Streamer can sign up using Kick OAuth |
| FR2 | Auth | Streamer can sign up using Twitch OAuth |
| FR3 | Auth | Streamer can connect additional platform after initial signup |
| FR4 | Auth | Streamer can disconnect a platform from their account |
| FR5 | Auth | Streamer can view all connected platforms in dashboard |
| **Overlay Layout Management** |||
| FR6 | Layouts | Streamer can create a new overlay layout |
| FR7 | Layouts | Streamer can drag-and-drop widgets onto layout grid |
| FR8 | Layouts | Streamer can reposition widgets within the layout |
| FR9 | Layouts | Streamer can remove widgets from layout |
| FR10 | Layouts | Streamer can save layout configuration |
| FR11 | Layouts | Streamer can create multiple saved layouts |
| FR12 | Layouts | Streamer can set a default layout |
| FR13 | Layouts | Streamer can duplicate an existing layout |
| FR14 | Layouts | Streamer can delete a layout |
| **Core Widgets** |||
| FR15 | Widgets | Streamer can add Chat widget to layout |
| FR16 | Widgets | Streamer can add Events widget to layout |
| FR17 | Widgets | Streamer can add Goals widget to layout |
| FR18 | Widgets | Streamer can add Tips widget to layout |
| FR19 | Widgets | Streamer can configure individual widget settings |
| **Theme Customization** |||
| FR20 | Themes | Streamer can customize global color scheme |
| FR21 | Themes | Streamer can customize fonts |
| FR22 | Themes | Streamer can override colors per widget |
| FR23 | Themes | Streamer can save theme configuration |
| FR24 | Themes | Streamer can export/import theme |
| **Public Overlay Access** |||
| FR25 | Overlay | System provides unique overlay URL per streamer per platform |
| FR26 | Overlay | Overlay URL renders layout in real-time for OBS |
| FR27 | Overlay | Overlay receives real-time chat messages via WebSocket |
| FR28 | Overlay | Overlay receives real-time events via WebSocket |
| FR29 | Overlay | Overlay updates goals in real-time from platform API |
| **Platform Integration** |||
| FR30 | Platform | System connects to Kick chat via WebSocket |
| FR31 | Platform | System connects to Twitch chat via IRC pattern |
| FR32 | Platform | System receives Kick webhooks for events |
| FR33 | Platform | System receives Twitch EventSub for events |
| FR34 | Platform | System fetches follower/subscriber counts from platform APIs |
| **Admin Dashboard** |||
| FR35 | Dashboard | Streamer can access admin dashboard after authentication |
| FR36 | Dashboard | Streamer can navigate between dashboard sections |
| FR37 | Dashboard | Streamer can view overlay preview in dashboard |
| **Data Isolation** |||
| FR38 | Isolation | System isolates all streamer data by streamer_id |
| FR39 | Isolation | Streamer cannot access another streamer's data |
| FR40 | Isolation | System cascades deletes when streamer account removed |

**Total: 40 Functional Requirements across 8 categories**

---

## FR Coverage Map

<!-- fr_coverage_map placeholder -->

---

## Epic 1: Foundation & Project Setup

**Goal:** Establish the technical foundation enabling all subsequent development. After this epic, developers have a working monorepo with database, local development environment, and core infrastructure patterns in place.

**User Value:** While not user-facing, this epic ensures data isolation (FR38-40) is enforced from day one, preventing security issues later.

**Technical Context:**
- Bun workspace monorepo structure
- Docker Compose for PostgreSQL 16 + Redis 7
- Drizzle ORM with tenant isolation patterns
- Next.js 15 App Router shell
- Elysia WebSocket server skeleton

---

### Story 1.1: Initialize Bun Workspace Monorepo

As a developer,
I want to initialize the Bun workspace monorepo structure,
So that all packages and apps can share dependencies and types.

**Acceptance Criteria:**

**Given** an empty project directory
**When** I run the initialization commands
**Then** the following structure is created:

```
streaming-system/
├── package.json          # workspaces: ["apps/*", "packages/*"]
├── bun.lockb
├── .gitignore
├── .env.example
├── apps/
│   ├── web/              # placeholder
│   └── ws-server/        # placeholder
├── packages/
│   ├── db/               # placeholder
│   ├── shared/           # placeholder
│   └── platform-adapters/# placeholder
```

**And** `bun install` succeeds without errors
**And** workspace references work between packages

**Technical Notes:**
- Use `bun init` at root
- Configure `workspaces` in root package.json (Architecture section: Monorepo Structure)
- Add `.gitignore` with node_modules, .env, dist, .next
- Create `.env.example` with placeholder variables

**Prerequisites:** None

---

### Story 1.2: Set Up Local Development Environment

As a developer,
I want Docker Compose configuration for local PostgreSQL and Redis,
So that I can develop without external dependencies.

**Acceptance Criteria:**

**Given** the monorepo from Story 1.1
**When** I run `docker-compose up -d`
**Then** PostgreSQL 16 is running on port 5432
**And** Redis 7 is running on port 6379
**And** a `streamvibes` database is created
**And** I can connect to both services from my application

**Technical Notes:**
- PostgreSQL 16 with `POSTGRES_DB=streamvibes` (Architecture: PostgreSQL 16)
- Redis 7 for pub/sub and caching (Architecture: Redis 7)
- Use Docker volumes for data persistence
- Include health checks in compose file

**Prerequisites:** Story 1.1

---

### Story 1.3: Create Database Schema Package

As a developer,
I want the Drizzle ORM schema package with core tables,
So that all apps can share type-safe database access.

**Acceptance Criteria:**

**Given** the monorepo with local database running
**When** I set up the `packages/db` package
**Then** the following tables are defined with Drizzle:

- `streamers` (id, email, display_name, avatar_url, created_at, updated_at)
- `platform_connections` (id, streamer_id, platform, platform_user_id, access_token, refresh_token, expires_at)
- `sessions` (id, streamer_id, token, expires_at)
- `layouts` (id, streamer_id, name, is_default, widget_positions JSONB, created_at, updated_at)
- `widgets` (id, streamer_id, layout_id, widget_type, config JSONB, position JSONB)
- `themes` (id, streamer_id, name, is_default, variables JSONB)

**And** every table has `streamer_id` column for tenant isolation (FR38)
**And** foreign keys cascade on delete (FR40)
**And** `drizzle-kit generate` creates migration SQL
**And** `drizzle-kit migrate` applies migrations successfully

**Technical Notes:**
- Use identity columns for IDs (Architecture: Data Architecture)
- Use snake_case for table and column names (Architecture: Naming Conventions)
- JSONB for flexible widget configs (Architecture: PostgreSQL 16)
- Export typed schema from `packages/db/index.ts`
- Include `@streamvibes/db` workspace reference

**Prerequisites:** Story 1.2

---

### Story 1.4: Create Shared Types Package

As a developer,
I want a shared types package for TypeScript interfaces and Zod schemas,
So that frontend and backend share the same type definitions.

**Acceptance Criteria:**

**Given** the database schema from Story 1.3
**When** I set up the `packages/shared` package
**Then** the following are exported:

**Types:**
- `Streamer`, `PlatformConnection`, `Layout`, `Widget`, `Theme`
- `WidgetType` enum (chat, events, goals, tips)
- `Platform` enum (kick, twitch)
- `WebSocketMessage` type with `{type, payload, timestamp, streamerId}`

**Zod Schemas:**
- `layoutSchema`, `widgetConfigSchema`, `themeSchema`
- Validation for all API inputs

**Constants:**
- `PLATFORMS`, `WIDGET_TYPES`
- `WS_RECONNECT_DELAY`, `MAX_WIDGETS`

**And** types can be imported as `@streamvibes/shared`
**And** types align with database schema

**Technical Notes:**
- Use Zod v3.x for runtime validation (Architecture: Validation)
- Export from `packages/shared/index.ts`
- camelCase for schema names with `Schema` suffix (Architecture: Naming Conventions)

**Prerequisites:** Story 1.3

---

### Story 1.5: Initialize Next.js Application Shell

As a developer,
I want the Next.js 15 application shell with App Router,
So that we have a foundation for the admin dashboard.

**Acceptance Criteria:**

**Given** the monorepo with packages set up
**When** I create the `apps/web` application
**Then** Next.js 15 is configured with:

- App Router structure (`app/` directory)
- TypeScript strict mode
- Tailwind CSS configured
- shadcn/ui initialized with base components
- `output: 'standalone'` in next.config.ts for Docker

**And** the following route groups exist:
- `app/(public)/` - for public pages
- `app/(dashboard)/` - for authenticated pages
- `app/api/` - for API routes

**And** `bun run dev` starts the development server
**And** the app renders a placeholder home page

**Technical Notes:**
- Use `bunx create-next-app` with App Router (Architecture: Frontend Framework)
- Configure `@streamvibes/shared` and `@streamvibes/db` as dependencies
- Set up path aliases in tsconfig.json
- Initialize shadcn/ui with default theme (UX: UI Framework)

**Prerequisites:** Story 1.4

---

### Story 1.6: Initialize Elysia WebSocket Server

As a developer,
I want the Elysia WebSocket server skeleton,
So that we have a foundation for real-time overlay updates.

**Acceptance Criteria:**

**Given** the monorepo with Next.js app
**When** I create the `apps/ws-server` application
**Then** Elysia v1.2 is configured with:

- Basic HTTP health check endpoint at `/health`
- WebSocket endpoint at `/ws`
- Redis connection for pub/sub
- Connection management service skeleton
- Message router skeleton

**And** the server starts on port 3001
**And** WebSocket connections can be established
**And** health check returns 200 OK

**Technical Notes:**
- Use Elysia v1.2 on Bun (Architecture: Real-time Server)
- Set up ioredis for pub/sub (Architecture: Redis)
- Create placeholder handlers in `src/handlers/`
- Export WebSocket message types from `@streamvibes/shared`

**Prerequisites:** Story 1.5

---

**Epic 1 Complete**

**Stories:** 6
**FR Coverage:** FR38 (isolation), FR39 (access control), FR40 (cascade deletes) - enforced through schema design
**Architecture Sections Used:** Monorepo Structure, Data Architecture, Naming Conventions, PostgreSQL 16, Redis 7, Frontend Framework, Real-time Server

---

## Epic 2: Streamer Authentication

**Goal:** Enable streamers to sign up using Kick or Twitch OAuth and manage their connected platforms. After this epic, streamers can create accounts and connect multiple streaming platforms.

**User Value:** Streamers can sign up with one click using their existing Kick or Twitch account, and later connect additional platforms.

**Technical Context:**
- Better Auth with Drizzle adapter
- Arctic OAuth providers for Kick and Twitch
- Database sessions stored in PostgreSQL
- Platform connection management

**UX Context:**
- OAuth signup flow (UX Flow 1: Streamer Signup)
- Platform connection in Settings (UX Flow 4: Add Second Platform)

---

### Story 2.1: Configure Better Auth with Drizzle

As a developer,
I want Better Auth configured with the Drizzle adapter,
So that authentication state is stored in our PostgreSQL database.

**Acceptance Criteria:**

**Given** the Next.js app and database schema from Epic 1
**When** I configure Better Auth
**Then** the following is set up:

- Better Auth initialized in `apps/web/lib/auth.ts`
- Drizzle adapter connected to `@streamvibes/db`
- Session table utilized for database sessions
- CSRF protection enabled
- Auth API routes at `/api/auth/[...betterauth]`

**And** the auth configuration exports:
- `auth` server instance
- `signIn`, `signOut` client functions
- `useSession` hook for client components

**Technical Notes:**
- Use Better Auth with Drizzle adapter (Architecture: Authentication & Security)
- Database sessions for tenant-scoped queries (Architecture: Session Strategy)
- Configure session expiry (7 days default)
- Set secure cookie options for production

**Prerequisites:** Epic 1 complete

---

### Story 2.2: Implement Kick OAuth Provider

As a streamer,
I want to sign up using my Kick account,
So that I can quickly create a StreamVibes account without a new password.

**Acceptance Criteria:**

**Given** I am on the login page
**When** I click "Sign in with Kick"
**Then** I am redirected to Kick's OAuth authorization page

**And** after I authorize the application
**Then** I am redirected back to StreamVibes
**And** a new `streamers` record is created with my Kick profile data
**And** a `platform_connections` record is created for Kick
**And** my OAuth tokens are encrypted and stored (NFR6)
**And** I am redirected to the dashboard

**Given** I already have an account
**When** I sign in with Kick
**Then** I am logged into my existing account
**And** my tokens are refreshed if needed

**Technical Notes:**
- Use Arctic Kick provider (Architecture: OAuth)
- Store `platform_user_id` for Kick
- Encrypt access/refresh tokens at rest (Architecture: Security)
- Handle token refresh automatically (NFR20)
- Rate limit auth endpoints (Architecture: Security Middleware)

**Prerequisites:** Story 2.1

---

### Story 2.3: Implement Twitch OAuth Provider

As a streamer,
I want to sign up using my Twitch account,
So that I can use StreamVibes with my Twitch channel.

**Acceptance Criteria:**

**Given** I am on the login page
**When** I click "Sign in with Twitch"
**Then** I am redirected to Twitch's OAuth authorization page

**And** after I authorize the application
**Then** I am redirected back to StreamVibes
**And** a new `streamers` record is created with my Twitch profile data
**And** a `platform_connections` record is created for Twitch
**And** my OAuth tokens are encrypted and stored (NFR6)
**And** I am redirected to the dashboard

**Given** I already have an account
**When** I sign in with Twitch
**Then** I am logged into my existing account
**And** my tokens are refreshed if needed

**Technical Notes:**
- Use Arctic Twitch provider (Architecture: OAuth)
- Request scopes: `user:read:email`, `channel:read:subscriptions`, `chat:read`
- Store `platform_user_id` for Twitch
- Encrypt access/refresh tokens at rest
- Handle Twitch-specific token refresh flow

**Prerequisites:** Story 2.1

---

### Story 2.4: Connect Additional Platform

As a streamer,
I want to connect a second streaming platform to my account,
So that I can manage both Kick and Twitch from one dashboard.

**Acceptance Criteria:**

**Given** I am logged in with Kick
**And** I am on the Settings page
**When** I click "Connect Twitch"
**Then** I am redirected to Twitch OAuth
**And** after authorization, a new `platform_connections` record is created
**And** my account now shows both platforms connected
**And** I return to Settings with a success message

**Given** I am logged in with Twitch
**And** I am on the Settings page
**When** I click "Connect Kick"
**Then** the same flow works for connecting Kick

**And** I cannot connect the same platform twice
**And** I see an error if I try to connect a platform already linked to another account

**Technical Notes:**
- Reuse Arctic OAuth flow with `connect` intent
- Link to existing `streamer_id` instead of creating new account
- Validate platform not already connected (FR3)
- Show connected platforms in Settings (FR5)

**Prerequisites:** Stories 2.2, 2.3

---

### Story 2.5: Disconnect Platform

As a streamer,
I want to disconnect a streaming platform from my account,
So that I can remove access if I no longer use that platform.

**Acceptance Criteria:**

**Given** I have both Kick and Twitch connected
**And** I am on the Settings page
**When** I click "Disconnect" next to Twitch
**Then** I see a confirmation dialog warning about data loss
**And** after confirming, the `platform_connections` record is deleted
**And** the platform no longer appears in my connected platforms

**Given** I only have one platform connected
**When** I try to disconnect it
**Then** I see an error "You must have at least one platform connected"
**And** the disconnect is prevented

**Technical Notes:**
- Soft validation: at least one platform required (FR4)
- Cascade related data appropriately
- Revoke OAuth tokens on disconnect if platform supports it
- Show confirmation dialog (UX best practice)

**Prerequisites:** Story 2.4

---

### Story 2.6: Auth Middleware and Session Handling

As a developer,
I want middleware that protects dashboard routes and provides session context,
So that all authenticated pages have access to the current streamer.

**Acceptance Criteria:**

**Given** a user without a valid session
**When** they try to access `/dashboard/*`
**Then** they are redirected to `/auth/login`

**Given** a user with a valid session
**When** they access `/dashboard/*`
**Then** the page renders with session context available
**And** `session.streamerId` is available for tenant-scoped queries

**And** the session includes:
- `streamerId` (for tenant isolation)
- `email`, `displayName`, `avatarUrl`
- `connectedPlatforms` array

**And** expired sessions redirect to login
**And** session refresh happens automatically

**Technical Notes:**
- Use Next.js middleware for route protection (Architecture: Project Structure)
- Better Auth session handling
- Provide `useSession()` hook for client components
- Server components use `auth()` directly
- Enforce tenant context in all queries (FR38, FR39)

**Prerequisites:** Stories 2.2, 2.3

---

**Epic 2 Complete**

**Stories:** 6
**FR Coverage:** FR1 (Kick OAuth), FR2 (Twitch OAuth), FR3 (connect additional), FR4 (disconnect), FR5 (view connected), FR35 (dashboard access - partially)
**Architecture Sections Used:** Authentication & Security, OAuth, Session Strategy, Security Middleware
**UX Sections Used:** Flow 1 (Streamer Signup), Flow 4 (Add Second Platform)

---

## Epic 3: Dashboard & Navigation

**Goal:** Provide streamers with a functional admin dashboard they can navigate to manage their overlays. After this epic, streamers have a home base with sidebar navigation and can move between dashboard sections.

**User Value:** Streamers can access their personal admin area and navigate between overlay editor, theme settings, widgets, and account settings.

**Technical Context:**
- Next.js App Router `(dashboard)` route group
- TanStack Query for data fetching
- shadcn/ui components for layout
- Protected routes with session context

**UX Context:**
- Dashboard layout components (Sidebar, Header, PlatformSwitcher)
- Navigation structure from UX Application Structure
- Desktop-first with tablet/mobile considerations

---

### Story 3.1: Dashboard Layout Shell

As a streamer,
I want a consistent dashboard layout with sidebar navigation,
So that I can easily navigate between different sections of the admin area.

**Acceptance Criteria:**

**Given** I am logged in
**When** I access `/dashboard`
**Then** I see a layout with:

- **Sidebar** (left): Navigation menu with icons and labels
- **Header** (top): My avatar, display name, and logout button
- **Main content area**: Page-specific content
- **Platform Switcher**: Toggle between Kick/Twitch views

**And** the sidebar contains navigation links to:
- Dashboard (home)
- Overlay Editor
- Widgets
- Theme
- Settings

**And** the current page is highlighted in the sidebar
**And** the layout is responsive (sidebar collapses on tablet)

**Technical Notes:**
- Create `app/(dashboard)/layout.tsx` (Architecture: Project Structure)
- Use shadcn/ui Sidebar component
- Implement Header with UserMenu component
- Use `usePathname()` for active link highlighting
- Mobile: hamburger menu toggle (UX: Responsive Behavior)

**Prerequisites:** Epic 2 complete (auth middleware)

---

### Story 3.2: Dashboard Home Page

As a streamer,
I want a dashboard home page showing my account overview,
So that I can quickly see my connected platforms and recent activity.

**Acceptance Criteria:**

**Given** I am logged in
**When** I navigate to `/dashboard`
**Then** I see:

- **Welcome message** with my display name
- **Connected Platforms** card showing Kick/Twitch status
- **Quick Stats** cards (layouts count, active widgets)
- **Quick Actions**: "Edit Overlay", "Customize Theme"
- **Overlay URL** card with copy button

**And** stats are fetched from the database with TanStack Query
**And** the page loads within 2 seconds (NFR2)
**And** empty states show helpful messages for new users

**Technical Notes:**
- Create `app/(dashboard)/dashboard/page.tsx`
- Use TanStack Query for data fetching (Architecture: State Management)
- Use shadcn/ui Card components for stat cards
- Include copy-to-clipboard for overlay URL
- Tenant-scoped queries with `session.streamerId`

**Prerequisites:** Story 3.1

---

### Story 3.3: Platform Switcher Component

As a streamer,
I want to switch between my Kick and Twitch views,
So that I can manage platform-specific settings and see platform-specific stats.

**Acceptance Criteria:**

**Given** I have both Kick and Twitch connected
**When** I click the Platform Switcher in the header
**Then** I see a dropdown with both platforms
**And** selecting a platform updates the current context
**And** the selected platform persists across navigation
**And** platform-specific data refreshes when switched

**Given** I only have one platform connected
**Then** the Platform Switcher shows only that platform
**And** no dropdown is shown (just the platform badge)

**And** the current platform is stored in:
- URL query param (shareable)
- Or localStorage (persistent)

**Technical Notes:**
- Create `components/layout/PlatformSwitcher.tsx`
- Use React context for current platform state
- Use shadcn/ui DropdownMenu
- Sync with TanStack Query for data refetching
- Consider URL param `?platform=kick` for deep linking

**Prerequisites:** Story 3.1

---

### Story 3.4: Settings Page with Connected Platforms

As a streamer,
I want a settings page where I can view and manage my connected platforms,
So that I can connect new platforms or disconnect existing ones.

**Acceptance Criteria:**

**Given** I am logged in
**When** I navigate to `/dashboard/settings`
**Then** I see:

- **Account section**: Email, display name, avatar
- **Connected Platforms section**: List of connected platforms with:
  - Platform icon and name
  - Connected date
  - "Disconnect" button (if more than one connected)
  - "Connect" button for unconnected platforms

**And** clicking "Connect Kick" or "Connect Twitch" initiates OAuth flow (Story 2.4)
**And** clicking "Disconnect" shows confirmation and removes connection (Story 2.5)
**And** I see FR5: all connected platforms displayed

**Technical Notes:**
- Create `app/(dashboard)/settings/page.tsx`
- Fetch platform connections with TanStack Query
- Reuse OAuth connection logic from Epic 2
- Use shadcn/ui Card for sections
- Show loading states during OAuth redirects

**Prerequisites:** Stories 3.1, 2.4, 2.5

---

**Epic 3 Complete**

**Stories:** 4
**FR Coverage:** FR35 (dashboard access), FR36 (navigation), FR37 (overlay preview - foundation only), FR5 (view connected platforms - in settings)
**Architecture Sections Used:** Project Structure, State Management (TanStack Query), Frontend Architecture
**UX Sections Used:** Dashboard Components, Application Structure, Responsive Behavior

---

## Epic 4: Layout Builder Core

**Goal:** Enable streamers to visually create and manage overlay layouts using drag-and-drop. After this epic, streamers can create layouts, position widgets on a grid, save their work, and manage multiple layouts.

**User Value:** Streamers can design custom overlay layouts visually without any coding, just like arranging elements in a design tool.

**Technical Context:**
- @dnd-kit for drag-and-drop functionality
- `layouts` table with JSONB for widget positions
- React Hook Form + Zod for validation
- TanStack Query for data mutations
- API routes: `/api/layouts/*`

**UX Context:**
- Grid Canvas (12x9 for 1920x1080 reference)
- Widget Handle, Widget Picker, Layout Selector
- Build Overlay flow (UX Flow 2)

---

### Story 4.1: Layout API Routes

As a developer,
I want REST API routes for layout CRUD operations,
So that the frontend can create, read, update, and delete layouts.

**Acceptance Criteria:**

**Given** an authenticated streamer
**When** API requests are made to `/api/layouts`
**Then** the following endpoints work:

| Method | Endpoint | Action |
|--------|----------|--------|
| GET | `/api/layouts` | List all layouts for current streamer |
| POST | `/api/layouts` | Create new layout |
| GET | `/api/layouts/[layoutId]` | Get single layout |
| PUT | `/api/layouts/[layoutId]` | Update layout |
| DELETE | `/api/layouts/[layoutId]` | Delete layout |

**And** all queries filter by `session.streamerId` (FR38, FR39)
**And** responses follow standard format: `{ data: T }` or `{ error: {...} }`
**And** validation uses Zod `layoutSchema`
**And** 403 returned if accessing another streamer's layout

**Technical Notes:**
- Create `app/api/layouts/route.ts` and `app/api/layouts/[layoutId]/route.ts`
- Use Drizzle for queries with `eq(layouts.streamerId, session.streamerId)`
- Validate request body with `layoutSchema` from `@streamvibes/shared`
- Return proper HTTP status codes (Architecture: API Response Patterns)

**Prerequisites:** Epic 3 complete

---

### Story 4.2: Layout Editor Page Shell

As a streamer,
I want to access the layout editor page,
So that I can start building my overlay.

**Acceptance Criteria:**

**Given** I am logged in
**When** I navigate to `/dashboard/overlay`
**Then** I see the Layout Editor with:

- **Layout Selector** (top): Dropdown to switch between saved layouts
- **Toolbar**: "New Layout", "Save", "Duplicate", "Delete" buttons
- **Grid Canvas** (center): 12x9 grid representing 1920x1080
- **Widget Picker** (side panel or modal): Available widgets to add
- **Settings Panel** (right): Configuration for selected widget

**And** if I have no layouts, a "Create Your First Layout" prompt appears
**And** clicking "New Layout" opens a name dialog

**Technical Notes:**
- Create `app/(dashboard)/overlay/page.tsx`
- Use TanStack Query to fetch layouts
- Layout state managed with React state + mutations
- Grid canvas sized to fit screen with aspect ratio preserved
- Use shadcn/ui Dialog for name input

**Prerequisites:** Story 4.1

---

### Story 4.3: Grid Canvas with Drag-and-Drop

As a streamer,
I want a visual grid where I can drag and drop widgets,
So that I can position elements exactly where I want them on my overlay.

**Acceptance Criteria:**

**Given** I am in the Layout Editor
**When** I view the Grid Canvas
**Then** I see a 12-column by 9-row grid (1920x1080 aspect ratio)
**And** grid lines are subtly visible for alignment

**When** I drag a widget from the Widget Picker
**Then** I can drop it onto the grid
**And** the widget snaps to grid cells
**And** the widget shows a visual placeholder during drag

**When** I drag an existing widget on the grid
**Then** I can reposition it to a new location (FR8)
**And** widgets cannot overlap (or show warning if they do)

**And** widgets can span multiple grid cells (resizable)
**And** the grid responds smoothly to interactions

**Technical Notes:**
- Use @dnd-kit for drag-and-drop (Architecture: Drag-and-Drop)
- Create `components/editor/GridCanvas.tsx`
- Create `components/editor/WidgetHandle.tsx` for draggable widgets
- Store positions as `{ x, y, width, height }` in grid units
- Use CSS Grid for layout, transform for drag preview

**Prerequisites:** Story 4.2

---

### Story 4.4: Widget Picker Modal

As a streamer,
I want to add widgets to my layout from a picker,
So that I can choose which elements appear on my overlay.

**Acceptance Criteria:**

**Given** I am in the Layout Editor
**When** I click "Add Widget" button
**Then** a modal opens showing available widgets:
- Chat Widget
- Events Widget (follows, subs, tips)
- Goals Widget
- Tips Widget

**And** each widget shows:
- Icon
- Name
- Brief description

**When** I click a widget in the picker
**Then** it is added to the canvas at a default position (FR7)
**And** the modal closes
**And** the new widget is selected

**And** I can add multiple widgets of the same type
**And** a maximum of 10 widgets per layout is enforced

**Technical Notes:**
- Create `components/editor/WidgetPicker.tsx`
- Use shadcn/ui Dialog and Card components
- Widget types from `@streamvibes/shared/constants`
- Trigger canvas update and optimistic UI

**Prerequisites:** Story 4.3

---

### Story 4.5: Remove Widget from Layout

As a streamer,
I want to remove widgets from my layout,
So that I can clean up elements I no longer want.

**Acceptance Criteria:**

**Given** I have widgets on my canvas
**When** I select a widget
**Then** I see a "Delete" button or trash icon

**When** I click delete
**Then** the widget is removed from the canvas (FR9)
**And** the layout state updates immediately
**And** I can undo the deletion (optional: with Ctrl+Z)

**And** alternatively, I can right-click a widget for context menu with "Delete"

**Technical Notes:**
- Add delete action to WidgetHandle component
- Update layout state to remove widget
- Consider implementing undo stack for better UX
- Use shadcn/ui ContextMenu for right-click option

**Prerequisites:** Story 4.4

---

### Story 4.6: Save Layout

As a streamer,
I want to save my layout changes,
So that my work is persisted and available for my overlay.

**Acceptance Criteria:**

**Given** I have made changes to my layout
**When** I click "Save"
**Then** the layout is saved to the database (FR10)
**And** I see a success toast notification
**And** the "unsaved changes" indicator disappears

**Given** I try to navigate away with unsaved changes
**Then** I see a confirmation dialog warning about losing changes

**And** save completes within 1 second
**And** validation errors show inline messages

**Technical Notes:**
- Use TanStack Query mutation for save
- Optimistic update for immediate feedback
- Validate with `layoutSchema` before save
- Implement `beforeunload` handler for unsaved changes
- Use shadcn/ui Toast for notifications

**Prerequisites:** Story 4.5

---

### Story 4.7: Multiple Layouts and Default Selection

As a streamer,
I want to create and manage multiple layouts,
So that I can have different overlay configurations for different stream types.

**Acceptance Criteria:**

**Given** I am in the Layout Editor
**When** I click "New Layout"
**Then** I can enter a name and create a new layout (FR6, FR11)
**And** the new layout is selected and empty

**When** I use the Layout Selector dropdown
**Then** I see all my saved layouts
**And** selecting one loads that layout onto the canvas

**When** I click "Set as Default" on a layout
**Then** that layout is marked as default (FR12)
**And** a star icon indicates the default layout

**When** I click "Duplicate"
**Then** a copy of the current layout is created (FR13)
**And** it's named "{original name} (Copy)"

**And** I cannot have more than 10 layouts (soft limit)

**Technical Notes:**
- Add `is_default` boolean to layouts table
- Only one layout can be default per streamer
- Use TanStack Query for layout list with caching
- Layout Selector uses shadcn/ui Select component

**Prerequisites:** Story 4.6

---

### Story 4.8: Delete Layout

As a streamer,
I want to delete layouts I no longer need,
So that I can keep my layout list organized.

**Acceptance Criteria:**

**Given** I have multiple layouts
**When** I click "Delete" on a layout
**Then** I see a confirmation dialog (FR14)
**And** after confirming, the layout is deleted
**And** associated widgets are cascade deleted
**And** I'm redirected to another layout (or empty state)

**Given** I only have one layout
**When** I try to delete it
**Then** I see a warning "You must have at least one layout"
**And** deletion is prevented

**Given** I delete the default layout
**Then** another layout automatically becomes default

**Technical Notes:**
- Use cascade delete in database (FR40)
- Prevent deleting last layout with API validation
- Auto-promote another layout to default if needed
- Use shadcn/ui AlertDialog for confirmation

**Prerequisites:** Story 4.7

---

**Epic 4 Complete**

**Stories:** 8
**FR Coverage:** FR6 (create layout), FR7 (drag-drop), FR8 (reposition), FR9 (remove widget), FR10 (save), FR11 (multiple layouts), FR12 (default), FR13 (duplicate), FR14 (delete)
**Architecture Sections Used:** API & Communication Patterns, Drag-and-Drop, State Management, API Response Patterns
**UX Sections Used:** Layout Editor Components, Flow 2 (Build Overlay)

---

## Epic 5: Widget System

**Goal:** Implement the core widgets (Chat, Events, Goals, Tips) that streamers can add to their layouts. After this epic, streamers can add all four widget types and configure their individual settings.

**User Value:** Streamers can populate their overlays with meaningful content - chat messages, event alerts, progress goals, and tip displays.

**Technical Context:**
- `widgets` table with per-widget config in JSONB
- Widget components for both editor preview and overlay rendering
- Widget settings panel with React Hook Form + Zod
- API routes: `/api/widgets/*`

**UX Context:**
- Overlay Components: Chat Widget, Events Widget, Goals Widget, Tips Widget
- Widget configuration panels
- Real-time preview in editor

---

### Story 5.1: Widget API Routes

As a developer,
I want REST API routes for widget CRUD operations,
So that widget configurations can be persisted and retrieved.

**Acceptance Criteria:**

**Given** an authenticated streamer
**When** API requests are made to `/api/widgets`
**Then** the following endpoints work:

| Method | Endpoint | Action |
|--------|----------|--------|
| GET | `/api/widgets?layoutId=X` | List widgets for a layout |
| POST | `/api/widgets` | Create new widget |
| GET | `/api/widgets/[widgetId]` | Get single widget |
| PUT | `/api/widgets/[widgetId]` | Update widget config |
| DELETE | `/api/widgets/[widgetId]` | Delete widget |

**And** all queries verify `streamer_id` ownership (FR38, FR39)
**And** widgets are linked to layouts via `layout_id`
**And** validation uses `widgetConfigSchema` per widget type
**And** responses follow standard API format

**Technical Notes:**
- Create `app/api/widgets/route.ts` and `app/api/widgets/[widgetId]/route.ts`
- Widget config stored as JSONB with type-specific schema
- Validate widget belongs to streamer's layout before operations
- Return proper HTTP status codes (Architecture: API Response Patterns)

**Prerequisites:** Epic 4 complete

---

### Story 5.2: Chat Widget Component

As a streamer,
I want to add a Chat widget to my overlay,
So that viewers can see chat messages on my stream.

**Acceptance Criteria:**

**Given** I am in the Layout Editor
**When** I add a Chat Widget to my layout (FR15)
**Then** the widget appears on the canvas with default styling

**When** I select the Chat Widget
**Then** the Settings Panel shows configuration options:
- Message display count (5-50)
- Animation style (slide, fade, none)
- Show badges (yes/no)
- Show timestamps (yes/no)
- Font size (small, medium, large)
- Message timeout (seconds before fade)

**And** changes preview in real-time on the canvas
**And** configuration is saved with the layout

**Technical Notes:**
- Create `components/widgets/ChatWidget.tsx`
- Editor version shows mock chat data
- Config schema: `chatWidgetConfigSchema`
- Store config in `widgets.config` JSONB
- Use CSS variables for theming integration

**Prerequisites:** Story 5.1

---

### Story 5.3: Events Widget Component

As a streamer,
I want to add an Events widget to my overlay,
So that viewers see follow, subscribe, and tip alerts.

**Acceptance Criteria:**

**Given** I am in the Layout Editor
**When** I add an Events Widget to my layout (FR16)
**Then** the widget appears on the canvas with default styling

**When** I select the Events Widget
**Then** the Settings Panel shows configuration options:
- Event types to show (follows, subs, tips - checkboxes)
- Animation style (bounce, slide, zoom, fade)
- Display duration (3-15 seconds)
- Sound enabled (yes/no) - future
- Alert size (compact, standard, large)
- Show profile pictures (yes/no)

**And** changes preview in real-time (with mock events)
**And** configuration is saved with the layout

**Technical Notes:**
- Create `components/widgets/EventsWidget.tsx`
- Editor shows rotating mock alerts
- Different templates for follow/sub/tip events
- Config schema: `eventsWidgetConfigSchema`
- Animations use CSS keyframes or Framer Motion

**Prerequisites:** Story 5.1

---

### Story 5.4: Goals Widget Component

As a streamer,
I want to add a Goals widget to my overlay,
So that viewers can see progress toward follower/subscriber targets.

**Acceptance Criteria:**

**Given** I am in the Layout Editor
**When** I add a Goals Widget to my layout (FR17)
**Then** the widget appears on the canvas with default styling

**When** I select the Goals Widget
**Then** the Settings Panel shows configuration options:
- Goal type (followers, subscribers, custom)
- Target value (number input)
- Current value (auto-fetched or manual)
- Display style (bar, circle, number only)
- Show percentage (yes/no)
- Label text (customizable)
- Bar/circle colors (use theme or custom)

**And** changes preview in real-time
**And** configuration is saved with the layout

**Technical Notes:**
- Create `components/widgets/GoalsWidget.tsx`
- Editor shows static mock progress
- Live overlay will fetch real values from platform API
- Config schema: `goalsWidgetConfigSchema`
- Support both visual bar and circular progress styles

**Prerequisites:** Story 5.1

---

### Story 5.5: Tips Widget Component

As a streamer,
I want to add a Tips widget to my overlay,
So that viewers see rotating tip/donation messages.

**Acceptance Criteria:**

**Given** I am in the Layout Editor
**When** I add a Tips Widget to my layout (FR18)
**Then** the widget appears on the canvas with default styling

**When** I select the Tips Widget
**Then** the Settings Panel shows configuration options:
- Rotation interval (5-60 seconds)
- Maximum tips to show (3-10)
- Display style (ticker, card, list)
- Show amounts (yes/no)
- Show timestamps (yes/no)
- Minimum amount to display ($1-$100)
- Animation style (fade, slide)

**And** changes preview in real-time (with mock tips)
**And** configuration is saved with the layout

**Technical Notes:**
- Create `components/widgets/TipsWidget.tsx`
- Editor shows rotating mock tip data
- Config schema: `tipsWidgetConfigSchema`
- Support ticker (scrolling), card (single), and list (stacked) modes

**Prerequisites:** Story 5.1

---

### Story 5.6: Widget Settings Panel

As a streamer,
I want a settings panel to configure selected widgets,
So that I can customize how each widget looks and behaves.

**Acceptance Criteria:**

**Given** I am in the Layout Editor
**When** I click on a widget in the canvas
**Then** the Settings Panel opens on the right (FR19)
**And** it shows the widget name and type at the top

**And** the panel contains:
- Widget-specific configuration fields (from Stories 5.2-5.5)
- "Apply" button to save changes
- "Reset to Defaults" button
- "Delete Widget" button

**When** I change a setting
**Then** the canvas preview updates in real-time
**And** changes are marked as "unsaved"

**When** I click "Apply"
**Then** the widget config is updated in state
**And** the settings panel shows success feedback

**Technical Notes:**
- Create `components/editor/SettingsPanel.tsx`
- Use React Hook Form with Zod validation
- Dynamic form based on widget type
- Use shadcn/ui form components (Input, Select, Switch, Slider)
- Debounce preview updates for performance

**Prerequisites:** Stories 5.2, 5.3, 5.4, 5.5

---

**Epic 5 Complete**

**Stories:** 6
**FR Coverage:** FR15 (chat widget), FR16 (events widget), FR17 (goals widget), FR18 (tips widget), FR19 (configure widgets)
**Architecture Sections Used:** API & Communication Patterns, Frontend Architecture, Component Architecture
**UX Sections Used:** Overlay Components, Widget configuration

---

## Epic 6: Theme Customization

**Goal:** Enable streamers to personalize their overlay appearance with colors, fonts, and per-widget overrides. After this epic, streamers can create a cohesive visual brand for their overlays.

**User Value:** Streamers can match their overlay to their brand identity without CSS knowledge - just pick colors and fonts.

**Technical Context:**
- `themes` table with CSS variables stored in JSONB
- ThemeProvider using CSS variable injection
- React Hook Form for theme editor
- API routes: `/api/themes/*`

**UX Context:**
- Theme Editor page with color pickers and font selectors
- Real-time preview (UX Flow 3: Customize Theme)
- Dynamic theming architecture from UX spec

---

### Story 6.1: Theme API Routes

As a developer,
I want REST API routes for theme CRUD operations,
So that theme configurations can be persisted and retrieved.

**Acceptance Criteria:**

**Given** an authenticated streamer
**When** API requests are made to `/api/themes`
**Then** the following endpoints work:

| Method | Endpoint | Action |
|--------|----------|--------|
| GET | `/api/themes` | List all themes for current streamer |
| POST | `/api/themes` | Create new theme |
| GET | `/api/themes/[themeId]` | Get single theme |
| PUT | `/api/themes/[themeId]` | Update theme |
| DELETE | `/api/themes/[themeId]` | Delete theme |

**And** all queries filter by `session.streamerId` (FR38, FR39)
**And** validation uses `themeSchema`
**And** responses follow standard API format

**Technical Notes:**
- Create `app/api/themes/route.ts` and `app/api/themes/[themeId]/route.ts`
- Theme variables stored as JSONB: `{ primary, secondary, accent, radius, fonts, ... }`
- Validate with `themeSchema` from `@streamvibes/shared`
- Return proper HTTP status codes

**Prerequisites:** Epic 3 complete

---

### Story 6.2: Theme Editor Page

As a streamer,
I want a theme editor page to customize my overlay colors and fonts,
So that I can create a cohesive visual brand.

**Acceptance Criteria:**

**Given** I am logged in
**When** I navigate to `/dashboard/theme`
**Then** I see the Theme Editor with:

- **Theme Selector**: Dropdown to switch between saved themes
- **Color Section**: Color pickers for primary, secondary, accent, background
- **Typography Section**: Font family selectors for headings and body
- **Spacing Section**: Border radius slider
- **Preview Panel**: Live preview of theme applied to sample widgets

**And** if I have no themes, a default theme is created
**And** changes preview in real-time (FR20, FR21)

**Technical Notes:**
- Create `app/(dashboard)/theme/page.tsx`
- Use TanStack Query to fetch themes
- Color pickers from shadcn/ui or react-colorful
- Font selector with Google Fonts subset
- Preview panel shows mini widget samples

**Prerequisites:** Story 6.1

---

### Story 6.3: Global Color Scheme Customization

As a streamer,
I want to customize the global color scheme,
So that all my widgets share a consistent look.

**Acceptance Criteria:**

**Given** I am on the Theme Editor page
**When** I use the color pickers
**Then** I can customize:
- **Primary Color**: Main accent color for buttons, highlights
- **Secondary Color**: Supporting color for backgrounds, borders
- **Accent Color**: Eye-catching color for alerts, notifications
- **Background Color**: Widget background color
- **Text Color**: Primary text color
- **Muted Text Color**: Secondary text color

**And** each color picker shows:
- Visual color swatch
- Hex input field
- HSL/RGB toggle (optional)

**And** changes apply to the preview immediately (FR20)
**And** color contrast warnings show if accessibility is poor

**Technical Notes:**
- Store colors in theme JSONB: `{ primary: "#...", secondary: "#...", ... }`
- Use CSS variables: `--primary`, `--secondary`, `--accent`, etc.
- Check WCAG contrast ratios for text colors
- Provide preset color palettes for quick selection

**Prerequisites:** Story 6.2

---

### Story 6.4: Font Customization

As a streamer,
I want to customize fonts for my overlay,
So that text matches my brand typography.

**Acceptance Criteria:**

**Given** I am on the Theme Editor page
**When** I use the font selectors
**Then** I can customize:
- **Heading Font**: Used for titles, usernames, alerts
- **Body Font**: Used for chat messages, descriptions
- **Font Size Scale**: Small/Medium/Large base size

**And** font options include:
- System fonts (fast loading)
- Popular Google Fonts subset (10-15 options)
- Preview of each font in dropdown

**And** changes apply to the preview immediately (FR21)
**And** fonts are loaded dynamically for preview

**Technical Notes:**
- Store fonts in theme JSONB: `{ headingFont: "Inter", bodyFont: "Roboto", ... }`
- Load Google Fonts via `next/font` or dynamic link injection
- Use CSS variables: `--font-heading`, `--font-body`
- Subset Google Fonts to keep bundle small

**Prerequisites:** Story 6.2

---

### Story 6.5: Per-Widget Color Overrides

As a streamer,
I want to override colors for individual widgets,
So that specific widgets can stand out from the global theme.

**Acceptance Criteria:**

**Given** I am in the Layout Editor with a widget selected
**When** I open the Settings Panel
**Then** I see an "Appearance" section with:
- "Use Theme Colors" toggle (default: on)
- Custom color pickers (shown when toggle is off)

**When** I disable "Use Theme Colors"
**Then** I can set custom colors for that widget only:
- Widget background color
- Widget text color
- Widget accent color

**And** the widget preview updates to show custom colors (FR22)
**And** custom colors are saved with the widget config

**Given** I re-enable "Use Theme Colors"
**Then** the widget reverts to global theme colors

**Technical Notes:**
- Add `colorOverrides` to widget config JSONB
- Widget components check for overrides before using theme variables
- Use CSS variable fallback: `var(--widget-bg, var(--background))`
- Merge override colors into widget style

**Prerequisites:** Stories 5.6, 6.3

---

### Story 6.6: Save Theme and Export/Import

As a streamer,
I want to save my theme and export/import theme configurations,
So that I can backup my settings or share with others.

**Acceptance Criteria:**

**Given** I have customized my theme
**When** I click "Save Theme"
**Then** the theme is saved to the database (FR23)
**And** I see a success notification

**When** I click "Export Theme"
**Then** a JSON file is downloaded containing my theme settings (FR24)
**And** the file is named `{theme-name}-theme.json`

**When** I click "Import Theme"
**Then** I can select a JSON file
**And** the theme settings are loaded and previewed
**And** I can save the imported theme as a new theme

**And** imported themes are validated before applying
**And** invalid files show clear error messages

**Technical Notes:**
- Export creates downloadable JSON blob
- Import uses file input with JSON validation
- Validate against `themeSchema` on import
- Create new theme record for imports (don't overwrite)

**Prerequisites:** Story 6.4

---

**Epic 6 Complete**

**Stories:** 6
**FR Coverage:** FR20 (global colors), FR21 (fonts), FR22 (per-widget override), FR23 (save theme), FR24 (export/import)
**Architecture Sections Used:** Frontend Architecture, Component Architecture, Data Architecture
**UX Sections Used:** Flow 3 (Customize Theme), Dynamic Theming Architecture

---

## Epic 7: Platform Adapters

**Goal:** Build the integration layer that connects to Kick and Twitch platforms for real-time chat and events. After this epic, the system can receive live chat messages, follows, subscriptions, and tips from both platforms.

**User Value:** Streamers' overlays display live data from their actual streaming platforms, not just mock data.

**Technical Context:**
- `packages/platform-adapters` package with adapter pattern
- Kick: WebSocket for chat, webhooks for events
- Twitch: IRC pattern for chat, EventSub for events
- Redis pub/sub for internal event distribution
- Platform API clients for fetching counts

**UX Context:**
- Data flows from platform → adapter → WebSocket server → overlay
- Overlay widgets consume real-time platform data

---

### Story 7.1: Platform Adapter Base Architecture

As a developer,
I want a base adapter pattern for platform integrations,
So that Kick and Twitch can share common interfaces while having platform-specific implementations.

**Acceptance Criteria:**

**Given** the `packages/platform-adapters` package
**When** I set up the base architecture
**Then** the following is created:

**Interfaces:**
- `PlatformAdapter` interface with methods:
  - `connect(channelId: string): Promise<void>`
  - `disconnect(): Promise<void>`
  - `onChatMessage(handler: ChatMessageHandler): void`
  - `onEvent(handler: EventHandler): void`
  - `fetchFollowerCount(): Promise<number>`
  - `fetchSubscriberCount(): Promise<number>`

**Types:**
- `ChatMessage`: `{ platform, channelId, userId, username, message, badges, timestamp }`
- `PlatformEvent`: `{ platform, channelId, type, data, timestamp }`
- `EventType`: `follow | subscription | tip | raid`

**Base Class:**
- `BaseAdapter` abstract class with shared utilities
- Connection state management
- Reconnection logic with exponential backoff (NFR13)
- Error handling and logging

**And** types are exported from `@streamvibes/platform-adapters`

**Technical Notes:**
- Create `packages/platform-adapters/src/base.ts`
- Export types from `packages/platform-adapters/index.ts`
- Use dependency injection for Redis client
- Implement reconnection with 1s, 2s, 4s, 8s... backoff (max 30s)

**Prerequisites:** Epic 1 complete

---

### Story 7.2: Kick Chat WebSocket Adapter

As a developer,
I want a Kick chat adapter that connects via WebSocket,
So that streamers can receive live Kick chat on their overlays.

**Acceptance Criteria:**

**Given** a Kick platform connection with valid tokens
**When** the adapter connects to a Kick channel
**Then** a WebSocket connection is established to Kick's chat system (FR30)
**And** incoming chat messages are parsed and normalized
**And** messages are published to Redis `chat:{streamerId}` channel
**And** connection state is tracked (connected/disconnected/reconnecting)

**Given** the WebSocket disconnects unexpectedly
**Then** the adapter automatically reconnects with exponential backoff (NFR13)
**And** a reconnection event is logged

**Given** a chat message is received
**Then** the following is extracted:
- User ID and display name
- Message content
- Badges (subscriber, moderator, VIP, etc.)
- Emotes (positions and IDs)
- Timestamp

**Technical Notes:**
- Create `packages/platform-adapters/src/kick/chat.ts`
- Use Kick's Pusher-based WebSocket (or native WS if available)
- Channel format: `chatrooms.{channel_id}`
- Parse message payload according to Kick API docs
- Handle rate limits gracefully (NFR19)

**Prerequisites:** Story 7.1

---

### Story 7.3: Twitch Chat IRC Adapter

As a developer,
I want a Twitch chat adapter using the IRC pattern,
So that streamers can receive live Twitch chat on their overlays.

**Acceptance Criteria:**

**Given** a Twitch platform connection with valid tokens
**When** the adapter connects to a Twitch channel
**Then** an IRC connection is established to `irc.chat.twitch.tv` (FR31)
**And** the adapter authenticates with OAuth token
**And** the adapter joins the channel `#{channel_name}`
**And** incoming messages are parsed and normalized
**And** messages are published to Redis `chat:{streamerId}` channel

**Given** the IRC connection disconnects
**Then** the adapter automatically reconnects (NFR13)
**And** re-joins the channel after authentication

**Given** a chat message is received
**Then** the following is extracted from IRC tags:
- User ID and display name
- Message content
- Badges (from `badges` tag)
- Emotes (from `emotes` tag)
- Color (from `color` tag)
- Timestamp

**Technical Notes:**
- Create `packages/platform-adapters/src/twitch/chat.ts`
- Use TMI.js or raw WebSocket to `wss://irc-ws.chat.twitch.tv`
- Request capabilities: `twitch.tv/membership`, `twitch.tv/tags`, `twitch.tv/commands`
- Parse IRC PRIVMSG format with tags
- Handle PING/PONG for keepalive

**Prerequisites:** Story 7.1

---

### Story 7.4: Kick Webhooks for Events

As a developer,
I want webhook handlers for Kick platform events,
So that streamers can receive follow and subscription alerts.

**Acceptance Criteria:**

**Given** the webhook endpoint is configured
**When** Kick sends a webhook for a follow event (FR32)
**Then** the signature is verified (NFR9)
**And** the event is normalized to `PlatformEvent` format
**And** the event is published to Redis `events:{streamerId}` channel
**And** a 200 response is returned to Kick

**When** Kick sends a webhook for a subscription event
**Then** the same flow applies with event type `subscription`

**When** Kick sends a webhook for a tip/donation event
**Then** the same flow applies with event type `tip`

**Given** an invalid signature
**Then** the webhook is rejected with 401
**And** the attempt is logged

**Given** webhook processing fails
**Then** the system returns 500 and Kick will retry (NFR15)

**Technical Notes:**
- Create `apps/web/app/api/webhooks/kick/route.ts`
- Verify HMAC-SHA256 signature from Kick
- Store webhook secret in environment variables
- Log all webhook payloads for debugging
- Use Zod to validate incoming payloads

**Prerequisites:** Story 7.1

---

### Story 7.5: Twitch EventSub for Events

As a developer,
I want Twitch EventSub integration for platform events,
So that streamers can receive follow and subscription alerts.

**Acceptance Criteria:**

**Given** a Twitch platform connection
**When** the streamer connects their Twitch account
**Then** EventSub subscriptions are created for:
- `channel.follow` (FR33)
- `channel.subscribe`
- `channel.subscription.gift`
- `channel.cheer` (bits)

**When** Twitch sends an EventSub notification
**Then** the signature is verified using HMAC-SHA256 (NFR9)
**And** challenge responses are handled for subscription verification
**And** events are normalized to `PlatformEvent` format
**And** events are published to Redis `events:{streamerId}` channel

**Given** an EventSub subscription fails or is revoked
**Then** the system re-subscribes automatically
**And** the failure is logged for monitoring

**Technical Notes:**
- Create `apps/web/app/api/webhooks/twitch/route.ts`
- Create EventSub subscription management service
- Handle `webhook_callback_verification` challenge
- Use Twitch EventSub secret for signature validation
- Consider WebSocket transport as alternative to webhooks

**Prerequisites:** Story 7.1

---

### Story 7.6: Platform API Clients for Counts

As a developer,
I want API clients that fetch follower and subscriber counts,
So that the Goals widget can display real progress.

**Acceptance Criteria:**

**Given** a valid platform connection with tokens
**When** the Kick API client fetches follower count (FR34)
**Then** the current follower count is returned
**And** the response is cached in Redis for 60 seconds

**When** the Kick API client fetches subscriber count
**Then** the current subscriber count is returned
**And** the response is cached in Redis for 60 seconds

**When** the Twitch API client fetches follower count (FR34)
**Then** the Helix API endpoint is called
**And** the count is returned and cached

**When** the Twitch API client fetches subscriber count
**Then** the Helix API endpoint is called
**And** the count is returned and cached

**Given** the OAuth token has expired
**Then** the token is refreshed automatically (NFR20)
**And** the request is retried with the new token

**Given** the platform API rate limit is hit
**Then** the request is delayed and retried (NFR19)
**And** the rate limit status is logged

**Technical Notes:**
- Create `packages/platform-adapters/src/kick/api.ts`
- Create `packages/platform-adapters/src/twitch/api.ts`
- Use Redis SET with TTL for caching
- Implement token refresh in base adapter
- Handle rate limits with `Retry-After` header

**Prerequisites:** Story 7.1

---

**Epic 7 Complete**

**Stories:** 6
**FR Coverage:** FR30 (Kick chat WebSocket), FR31 (Twitch chat IRC), FR32 (Kick webhooks), FR33 (Twitch EventSub), FR34 (follower/subscriber counts)
**Architecture Sections Used:** Platform Adapters, Integration Patterns, Real-time Event Flow, Redis Pub/Sub
**UX Sections Used:** Data flows to overlay widgets

---

## Epic 8: Live Overlay Rendering

**Goal:** Create the public overlay pages that render in OBS and display real-time data from WebSocket connections. After this epic, streamers have working overlay URLs they can add to OBS.

**User Value:** Streamers paste a URL into OBS and immediately see their custom overlay with live chat, events, and goals updating in real-time.

**Technical Context:**
- Public routes: `/:platform/:channel/overlay`, `/chat`, `/events`, `/goals`
- WebSocket client connecting to Elysia WS server
- Client-side rendering optimized for OBS browser source
- Transparent backgrounds, 1920x1080 fixed resolution
- Theme injection via CSS variables

**UX Context:**
- Overlay pages from UX Application Structure (Public Pages)
- OBS browser source compatibility
- Real-time widget updates

---

### Story 8.1: Public Overlay URL Structure

As a streamer,
I want a unique overlay URL for each of my connected platforms,
So that I can add platform-specific overlays to OBS.

**Acceptance Criteria:**

**Given** I am a registered streamer with Kick connected
**When** I access my overlay URL (FR25)
**Then** the URL format is `/{platform}/{channel}/overlay`
**And** for my Kick channel, it's `/kick/my-channel/overlay`

**Given** I have both Kick and Twitch connected
**Then** I have two overlay URLs:
- `/kick/my-kick-channel/overlay`
- `/twitch/my-twitch-channel/overlay`

**And** the overlay URLs are shown in my dashboard
**And** I can copy the URL with a single click
**And** the URLs are publicly accessible (no auth required)
**And** invalid channels return a styled 404 page

**Technical Notes:**
- Create `app/(public)/[platform]/[channel]/overlay/page.tsx`
- Use dynamic route segments
- Validate platform is `kick` or `twitch`
- Look up channel in `platform_connections` table
- Return 404 if channel not found or inactive

**Prerequisites:** Epic 7 complete

---

### Story 8.2: Overlay WebSocket Connection

As a developer,
I want the overlay page to connect to the WebSocket server,
So that it receives real-time chat and event updates.

**Acceptance Criteria:**

**Given** the overlay page loads
**When** the page initializes
**Then** a WebSocket connection is established to the Elysia WS server
**And** the connection subscribes to the channel's events
**And** the connection handles authentication via overlay token

**Given** a chat message arrives via WebSocket (FR27)
**Then** the message is added to the Chat widget
**And** rendering happens within 100ms (NFR1)

**Given** an event arrives via WebSocket (FR28)
**Then** the event triggers the Events widget
**And** the alert animation plays immediately

**Given** the WebSocket disconnects
**Then** the overlay automatically reconnects (NFR13)
**And** a subtle "reconnecting" indicator may appear
**And** messages are not duplicated on reconnection

**Technical Notes:**
- Create `lib/overlay-socket.ts` WebSocket client
- Use reconnecting-websocket or custom reconnection logic
- Subscribe message: `{ type: "subscribe", channelId, platform }`
- Handle message types: `chat`, `event`, `goal_update`
- Debounce reconnection attempts

**Prerequisites:** Story 8.1, Stories 7.2-7.5

---

### Story 8.3: Overlay Page Rendering

As a streamer,
I want my overlay to render my saved layout with proper styling,
So that it looks exactly how I designed it in the editor.

**Acceptance Criteria:**

**Given** my overlay URL is loaded
**When** the page renders (FR26)
**Then** my default layout is fetched and rendered
**And** all widgets are positioned according to my layout grid
**And** my theme colors and fonts are applied via CSS variables
**And** the page has a transparent background (for OBS)
**And** the page is fixed at 1920x1080 pixels

**And** the overlay shows:
- Chat Widget at my configured position
- Events Widget at my configured position
- Goals Widget at my configured position
- Tips Widget at my configured position

**And** each widget uses its saved configuration
**And** per-widget color overrides are applied

**Technical Notes:**
- Fetch layout and theme for streamer/platform combo
- Inject theme as CSS variables in `<style>` tag
- Use absolute positioning based on grid units
- Set `background: transparent` for OBS compatibility
- Meta viewport: `width=1920, height=1080, initial-scale=1`

**Prerequisites:** Story 8.2, Epic 6

---

### Story 8.4: Individual Widget Overlay URLs

As a streamer,
I want individual overlay URLs for each widget type,
So that I can add single widgets to different OBS scenes.

**Acceptance Criteria:**

**Given** I want to use widgets separately
**Then** I have URLs for each widget type:
- `/{platform}/{channel}/chat` - Chat only
- `/{platform}/{channel}/events` - Events only
- `/{platform}/{channel}/goals` - Goals only
- `/{platform}/{channel}/tips` - Tips only (future)

**When** I load the chat widget URL
**Then** only the Chat widget renders
**And** it uses my theme and chat widget configuration
**And** it receives real-time chat via WebSocket

**When** I load the events widget URL
**Then** only the Events widget renders
**And** it uses my theme and events widget configuration
**And** it receives real-time events via WebSocket

**And** widget URLs have transparent backgrounds
**And** widget URLs use appropriate sizing for the widget type

**Technical Notes:**
- Create separate routes: `app/(public)/[platform]/[channel]/chat/page.tsx`, etc.
- Reuse widget components from editor
- Connect to same WebSocket server
- Allow custom sizing via query params: `?width=400&height=600`

**Prerequisites:** Story 8.3

---

### Story 8.5: Real-Time Goal Updates

As a streamer,
I want my Goals widget to update in real-time,
So that viewers see progress toward my follower/subscriber goals.

**Acceptance Criteria:**

**Given** I have a Goals widget on my overlay
**When** the overlay loads
**Then** the current follower/subscriber count is fetched (FR29)
**And** the goal progress bar displays accurately

**Given** a new follower event occurs
**Then** the goal count increments immediately
**And** the progress bar animates to the new value
**And** if the goal is reached, a celebration animation plays

**Given** the goal type is set to "subscribers"
**Then** the count reflects current subscriber count
**And** updates happen when subscription events occur

**And** goal values are polled every 60 seconds as backup
**And** the widget handles API errors gracefully

**Technical Notes:**
- Initial count fetched from platform API (Story 7.6)
- Real-time updates via WebSocket `goal_update` messages
- Fallback polling using TanStack Query with refetchInterval
- Animate progress with CSS transitions
- Cache counts in Redis to reduce API calls

**Prerequisites:** Story 8.3, Story 7.6

---

**Epic 8 Complete**

**Stories:** 5
**FR Coverage:** FR25 (unique overlay URL), FR26 (real-time rendering), FR27 (chat via WebSocket), FR28 (events via WebSocket), FR29 (goal updates)
**Architecture Sections Used:** Real-time Event Flow, WebSocket Architecture, Public Page Routes
**UX Sections Used:** Public Pages, Overlay Components

---

## FR Coverage Matrix

| FR | Description | Epic | Stories |
|----|-------------|------|---------|
| FR1 | Kick OAuth signup | 2 | 2.2 |
| FR2 | Twitch OAuth signup | 2 | 2.3 |
| FR3 | Connect additional platform | 2 | 2.4 |
| FR4 | Disconnect platform | 2 | 2.5 |
| FR5 | View connected platforms | 2, 3 | 2.4, 3.4 |
| FR6 | Create new layout | 4 | 4.7 |
| FR7 | Drag-drop widgets | 4 | 4.3, 4.4 |
| FR8 | Reposition widgets | 4 | 4.3 |
| FR9 | Remove widgets | 4 | 4.5 |
| FR10 | Save layout | 4 | 4.6 |
| FR11 | Multiple layouts | 4 | 4.7 |
| FR12 | Set default layout | 4 | 4.7 |
| FR13 | Duplicate layout | 4 | 4.7 |
| FR14 | Delete layout | 4 | 4.8 |
| FR15 | Chat widget | 5 | 5.2 |
| FR16 | Events widget | 5 | 5.3 |
| FR17 | Goals widget | 5 | 5.4 |
| FR18 | Tips widget | 5 | 5.5 |
| FR19 | Configure widgets | 5 | 5.6 |
| FR20 | Global colors | 6 | 6.3 |
| FR21 | Customize fonts | 6 | 6.4 |
| FR22 | Per-widget overrides | 6 | 6.5 |
| FR23 | Save theme | 6 | 6.6 |
| FR24 | Export/import theme | 6 | 6.6 |
| FR25 | Unique overlay URL | 8 | 8.1 |
| FR26 | Real-time rendering | 8 | 8.3 |
| FR27 | Chat via WebSocket | 8 | 8.2 |
| FR28 | Events via WebSocket | 8 | 8.2 |
| FR29 | Goal updates | 8 | 8.5 |
| FR30 | Kick chat WebSocket | 7 | 7.2 |
| FR31 | Twitch chat IRC | 7 | 7.3 |
| FR32 | Kick webhooks | 7 | 7.4 |
| FR33 | Twitch EventSub | 7 | 7.5 |
| FR34 | Follower/sub counts | 7 | 7.6 |
| FR35 | Dashboard access | 2, 3 | 2.6, 3.1 |
| FR36 | Dashboard navigation | 3 | 3.1 |
| FR37 | Overlay preview | 3 | 3.2 |
| FR38 | Data isolation | 1 | 1.3 |
| FR39 | Access control | 1 | 1.3 |
| FR40 | Cascade deletes | 1 | 1.3 |

**All 40 Functional Requirements covered across 8 Epics and 46 Stories.**

---

## Epic Dependency Graph

```
Epic 1: Foundation
    ↓
Epic 2: Authentication ←──────────────────┐
    ↓                                     │
Epic 3: Dashboard                         │
    ↓                                     │
┌───┴───┬───────────────┐                 │
↓       ↓               ↓                 │
Epic 4  Epic 5          Epic 6            │
Layouts Widgets         Themes            │
    └───┬───┘               │             │
        ↓                   │             │
    Epic 7: Platform Adapters             │
        ↓                   │             │
    Epic 8: Live Overlay ←──┘ ────────────┘
```

**Implementation Order:**
1. Epic 1 (Foundation) - Must be first
2. Epic 2 (Authentication) - Depends on Epic 1
3. Epic 3 (Dashboard) - Depends on Epic 2
4. Epics 4, 5, 6 (Layouts, Widgets, Themes) - Can be parallelized after Epic 3
5. Epic 7 (Platform Adapters) - Can start after Epic 1, but practical after Epic 5
6. Epic 8 (Live Overlay) - Final epic, depends on all others
