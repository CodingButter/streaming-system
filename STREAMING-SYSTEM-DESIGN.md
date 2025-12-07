# StreamVibes - Multi-Platform Streaming Overlay System

## Design Document for BMAD Greenfield Development

**Version:** 1.0
**Date:** 2025-12-07
**Author:** CodingButter
**Status:** Ready for BMAD Kickoff

---

## Executive Summary

StreamVibes is a multi-tenant SaaS platform that provides customizable stream overlays, chat integration, viewer engagement features, and channel management tools for content creators on Kick and Twitch. The system allows streamers to authenticate with their streaming platforms, customize their overlay experience, and engage their community through interactive features.

---

## Vision Statement

**"Empower streamers with a modular, extensible platform that grows with their community."**

StreamVibes transforms the streaming experience by providing:
- Beautiful, customizable overlays that work across platforms
- Engaging viewer features that build community
- A plugin architecture that lets streamers enable only what they need
- Bank-vault level data isolation between streamers

---

## Problem Statement

### Current Pain Points for Streamers

1. **Platform Lock-in**: Most tools only work with one platform (Twitch OR Kick)
2. **Feature Bloat**: Tools come with everything enabled, overwhelming new streamers
3. **No Customization**: Limited ability to customize look, feel, and behavior
4. **Expensive**: Enterprise tools charge per feature, costs add up quickly
5. **Poor Multi-Streaming Support**: Simulcasters have no unified experience

### Our Solution

A modular, multi-platform streaming system where:
- Streamers authenticate with Kick and/or Twitch
- Core features are always available (chat, events, goals, tips)
- Advanced features are opt-in plugins (Drop Game, AI Chatbot, Polls)
- Each streamer gets their own isolated data space
- One admin panel manages all connected platforms

---

## Target Users

### Primary: Small to Medium Streamers
- 10-1000 concurrent viewers
- Want professional overlays without complexity
- May stream on one or multiple platforms
- Budget-conscious, prefer freemium model

### Secondary: Simulcasters
- Stream to Kick AND Twitch simultaneously
- Need unified chat/event displays
- Want combined leaderboards or separate per platform

### Tertiary: Community Managers
- Help manage streamer's channel
- Need limited admin access
- Configure features on behalf of streamer

---

## Core Features (Always Enabled)

### 1. Overlay System

#### Chat Widget
- Display chat messages from connected platforms
- Platform badge/icon per message
- Text-to-Speech (TTS) support with 100+ AI voices
- Configurable message appearance
- Optional: Combined chat view for simulcasters

#### Events Widget
- Follow notifications
- Subscription notifications (new, resub, gift)
- Tip/donation alerts
- Raid/host notifications
- Custom event styling per platform

#### Goals Widget
- Follower goal progress bar
- Subscriber goal progress bar
- Custom goals (tips, etc.)
- Real-time updates from platform APIs

#### Tips Widget
- Rotating tip messages/announcements
- Configurable rotation speed
- Enable/disable individual tips
- Drag-and-drop reordering

#### Utility Widgets
- Chroma Key (solid color for green screen)
- Placeholder (empty space management)
- Custom HTML/CSS widget (advanced)

### 2. Layout Editor

- Multiple saved layouts per streamer
- Drag-and-drop widget placement
- Grid-based positioning
- Responsive scaling options
- Per-layout widget configuration
- Default layout selection
- Layout duplication

### 3. Theme System

- Global color customization
- Per-widget color overrides
- Font selection
- Border/shadow controls
- Import/export themes
- Community theme sharing (future)

### 4. Custom Commands

Streamers can create their own chat commands:
- `!socials` - Display social media links
- `!discord` - Show Discord invite
- `!schedule` - Show stream schedule
- Variables: `{username}`, `{platform}`, `{points}`
- Cooldowns per command
- Permission levels (everyone, subscribers, mods)

### 5. Viewer Profiles

- Per-platform viewer identity (viewer@kick vs viewer@twitch)
- Customizable TTS voice per viewer
- Custom drop game avatar
- Country flag display
- Profile page for viewers to manage settings

---

## Plugin System (Opt-In Features)

### Architecture Philosophy

Streamers start with core features only. They browse a "Plugin Store" in their admin panel and enable features they want. Each plugin adds:
- New chat commands
- New overlay widgets (optional)
- New admin configuration pages
- Database tables (auto-created)

### Plugin: Drop Game

**Description:** Interactive falling game where viewers compete for points

**Components Added:**
- `!drop` command to join game
- Drop Game overlay widget
- Powerup system (TNT, Shield, Magnet, Ghost, Boost)
- Powerup shop commands
- Leaderboard system
- Admin config: game timing, scoring, powerup costs

**Why Plugin:** Not all streamers want gamification. Some prefer chat-only interaction.

### Plugin: AI Chatbot

**Description:** LLM-powered chatbot that responds to viewers

**Components Added:**
- Trigger word/command configuration
- System prompt editor
- API key management (OpenAI, Anthropic, Google)
- Response settings (length, tone, cooldown)
- Context memory (optional recent messages)
- Admin config page for all settings

**Why Plugin:** Requires API keys, costs money per message, not everyone wants AI.

### Plugin: Channel Points Economy (Future)

**Description:** Unified points system across activities

**Components Added:**
- Points per chat message
- Points per watch time
- Points redemptions
- Custom rewards
- Points gambling commands

### Plugin: Polls & Predictions (Future)

**Description:** Interactive voting and betting

**Components Added:**
- `!poll` command for mods
- Poll overlay widget
- Prediction system with points betting
- Results display

### Plugin: Sound Alerts (Future)

**Description:** Custom sound effects triggered by events

**Components Added:**
- Sound library management
- Trigger configuration
- Volume controls
- Alert queue

---

## Technical Architecture

### Technology Stack

| Component | Technology | Rationale |
|-----------|------------|-----------|
| Runtime | Bun | Fast, built-in SQLite, TypeScript native |
| Backend | Bun.serve() | Lightweight, WebSocket support, no Express needed |
| Database | SQLite (bun:sqlite) | Simple, file-based, perfect for this scale |
| Frontend | React 18 | Component-based, large ecosystem |
| Styling | CSS-in-JS / Tailwind | Rapid development, theme support |
| State | React Context + hooks | Simple, no Redux overhead |
| TTS | ElevenLabs API | High-quality AI voices |
| Auth | OAuth 2.0 | Platform standard for Kick/Twitch |

### Multi-Tenant Data Model

```
┌─────────────────────────────────────────────────────────────────┐
│                         STREAMERS                                │
│  (Primary Tenant - Every other table references this)           │
├─────────────────────────────────────────────────────────────────┤
│  id | username | display_name | avatar_url | setup_completed    │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ PLATFORM_CONNS  │ │    VIEWERS      │ │ STREAMER_CONFIG │
│                 │ │                 │ │                 │
│ streamer_id     │ │ streamer_id     │ │ streamer_id     │
│ platform        │ │ platform        │ │ (theme, layout, │
│ platform_user_id│ │ username        │ │  settings, etc) │
│ access_token    │ │ voice_id        │ │                 │
│ refresh_token   │ │ drop_image      │ │                 │
└─────────────────┘ └─────────────────┘ └─────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  VIEWER_POINTS  │ │ VIEWER_INVENTORY│ │  VIEWER_HISTORY │
│                 │ │                 │ │                 │
│ viewer_id       │ │ viewer_id       │ │ viewer_id       │
│ channel_points  │ │ powerup_type    │ │ drops, purchases│
│ drop_points     │ │ quantity        │ │ transactions    │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### URL Structure

```
Public Routes:
  /                          → Marketing landing page
  /login                     → Platform selection (Kick/Twitch)
  /callback                  → OAuth callback handler

Streamer Admin (authenticated):
  /:username/admin           → Admin dashboard
  /:username/admin/overlay   → Layout editor
  /:username/admin/theme     → Theme customization
  /:username/admin/commands  → Custom commands
  /:username/admin/plugins   → Plugin store
  /:username/admin/[plugin]  → Plugin-specific config

Platform-Specific Public:
  /kick/:channel/overlay     → Kick overlay (OBS browser source)
  /kick/:channel/chat        → Kick chat-only widget
  /twitch/:channel/overlay   → Twitch overlay
  /twitch/:channel/chat      → Twitch chat-only widget

Viewer Pages:
  /kick/:channel/profile/:viewer     → Viewer profile (Kick)
  /twitch/:channel/profile/:viewer   → Viewer profile (Twitch)
  /kick/:channel/leaderboard         → Channel leaderboard
  /kick/:channel/commands            → Available commands

API Routes:
  /api/:platform/:channel/*          → Platform-scoped API
  /api/admin/:username/*             → Admin API (authenticated)
  /webhook/kick                      → Kick webhook receiver
  /webhook/twitch                    → Twitch EventSub receiver
```

### Security Model

#### Tenant Isolation (Bank-Vault Level)
- Every database query includes `WHERE streamer_id = ?`
- No query can access another streamer's data
- Foreign keys cascade deletes
- Tests verify isolation for every operation

#### Authentication
- Streamers: OAuth with Kick or Twitch (no passwords)
- Viewers: Verification code in chat (no passwords)
- Admin sessions: 7-day expiry, httpOnly cookies
- Viewer sessions: 24-hour expiry

#### API Security
- Rate limiting per IP and per user
- CORS restricted to known origins
- Webhook signature verification
- No sensitive data in URLs

---

## Platform Integration

### Kick Integration

| Feature | Implementation |
|---------|----------------|
| OAuth | Kick OAuth 2.0 |
| Chat | Kick Websocket API |
| Events | Kick Webhooks |
| User Data | Kick API |

### Twitch Integration

| Feature | Implementation |
|---------|----------------|
| OAuth | Twitch OAuth 2.0 |
| Chat | Twitch IRC (tmi.js pattern) |
| Events | Twitch EventSub |
| User Data | Twitch Helix API |

### Platform Adapter Pattern

```typescript
interface PlatformAdapter {
  // Identity
  readonly platform: 'kick' | 'twitch';

  // OAuth
  getAuthUrl(state: string): string;
  exchangeCode(code: string): Promise<TokenSet>;
  refreshToken(refreshToken: string): Promise<TokenSet>;

  // User Data
  getUser(accessToken: string): Promise<PlatformUser>;
  getFollowers(channelId: string): Promise<number>;
  getSubscribers(channelId: string): Promise<number>;

  // Chat
  connectChat(channelId: string, onMessage: ChatHandler): void;
  sendMessage(channelId: string, message: string): Promise<void>;

  // Webhooks
  verifyWebhook(request: Request): boolean;
  parseWebhookEvent(request: Request): WebhookEvent;
}
```

---

## User Flows

### Streamer Onboarding

```
1. Visit landing page → Click "Get Started"
2. Select platform (Kick or Twitch)
3. OAuth redirect → Authorize app
4. Return to app → Auto-create streamer account
5. Setup wizard:
   a. Confirm username/display name
   b. Choose initial theme
   c. Enable/disable default features
   d. Add overlay to OBS (copy URL)
6. Dashboard → Ready to stream!
```

### Adding Second Platform

```
1. Admin → Settings → Connected Platforms
2. Click "Connect Twitch" (or Kick)
3. OAuth flow for second platform
4. Platforms now show combined in admin
5. Separate overlays: /kick/... and /twitch/...
6. Optional: Enable combined chat widget
```

### Viewer Claiming Profile

```
1. Visit /:platform/:channel/profile/:username
2. See "Claim this profile" button
3. Get verification code
4. Type code in streamer's chat
5. Bot verifies → Profile unlocked
6. Customize voice, avatar, country
```

### Enabling a Plugin

```
1. Admin → Plugins
2. Browse available plugins
3. Click plugin → See description, features added
4. Click "Enable"
5. Plugin tables created, commands activated
6. New admin page appears in sidebar
7. Configure plugin settings
```

---

## Data Entities

### Core Entities

| Entity | Description |
|--------|-------------|
| Streamer | The tenant/channel owner |
| PlatformConnection | Links streamer to Kick/Twitch account |
| Viewer | A chat user (scoped to streamer+platform) |
| ViewerPoints | Points balance for a viewer |
| OverlayLayout | Saved layout configuration |
| OverlaySetting | Key-value settings |
| Theme | Color/style configuration |
| CustomCommand | Streamer-defined commands |
| Tip | Rotating tip messages |
| Goal | Progress bar goals |

### Plugin Entities (Drop Game)

| Entity | Description |
|--------|-------------|
| PowerupConfig | Powerup costs/settings |
| ViewerInventory | Owned powerups |
| DropHistory | Game results |
| PurchaseHistory | Shop transactions |

### Plugin Entities (AI Chatbot)

| Entity | Description |
|--------|-------------|
| ChatbotConfig | LLM settings, prompts |
| ChatbotConversation | Context memory |

---

## API Design

### RESTful Conventions

```
GET    /api/kick/:channel/viewers           → List viewers
GET    /api/kick/:channel/viewers/:id       → Get viewer
PATCH  /api/kick/:channel/viewers/:id       → Update viewer

GET    /api/admin/:username/layouts         → List layouts
POST   /api/admin/:username/layouts         → Create layout
GET    /api/admin/:username/layouts/:id     → Get layout
PUT    /api/admin/:username/layouts/:id     → Update layout
DELETE /api/admin/:username/layouts/:id     → Delete layout

GET    /api/admin/:username/plugins         → List available plugins
POST   /api/admin/:username/plugins/:id     → Enable plugin
DELETE /api/admin/:username/plugins/:id     → Disable plugin
```

### WebSocket Events

```typescript
// Client → Server
{ type: 'subscribe', channel: 'kick/codingbutter' }
{ type: 'unsubscribe', channel: 'kick/codingbutter' }

// Server → Client
{ type: 'chat', platform: 'kick', message: {...} }
{ type: 'follow', platform: 'kick', user: {...} }
{ type: 'subscription', platform: 'twitch', data: {...} }
{ type: 'drop_start', players: [...] }
{ type: 'drop_result', scores: [...] }
{ type: 'points_update', viewer: {...}, points: {...} }
```

---

## Non-Functional Requirements

### Performance
- Page load < 2 seconds
- Overlay updates < 100ms latency
- Support 1000 concurrent viewers per streamer
- Database queries < 50ms

### Availability
- 99.9% uptime target
- Graceful degradation if platform API down
- Webhook retry with exponential backoff

### Scalability
- Start: Single server, SQLite
- Growth: Can migrate to PostgreSQL + Redis
- Future: Horizontal scaling with read replicas

### Security
- OWASP Top 10 compliance
- No SQL injection (prepared statements)
- No XSS (React escaping + CSP)
- Encrypted tokens at rest

---

## Development Phases

### Phase 1: Foundation (MVP)
- [ ] Project setup with Bun
- [ ] SQLite database with multi-tenant schema
- [ ] Kick OAuth integration
- [ ] Basic overlay with chat widget
- [ ] Admin dashboard shell
- [ ] Single layout support

### Phase 2: Core Features
- [ ] Full widget suite (events, goals, tips)
- [ ] Theme customization
- [ ] Layout editor
- [ ] Multiple saved layouts
- [ ] Viewer profiles
- [ ] Custom commands

### Phase 3: Twitch Integration
- [ ] Twitch OAuth
- [ ] Twitch EventSub
- [ ] Platform adapter pattern
- [ ] Combined chat option
- [ ] Per-platform overlays

### Phase 4: Plugin System
- [ ] Plugin architecture
- [ ] Plugin store UI
- [ ] Drop Game as plugin
- [ ] AI Chatbot as plugin
- [ ] Plugin enable/disable

### Phase 5: Polish & Launch
- [ ] Marketing landing page
- [ ] Documentation
- [ ] Onboarding flow
- [ ] Performance optimization
- [ ] Public launch

---

## Reference Implementation

This design document is based on learnings from the `kick-redirect` project, which implemented many of these features in a single-tenant context. Key files to reference:

### Database Layer
- `server/data/types.ts` - Type definitions
- `server/data/interface.ts` - DataStore interface
- `server/data/sqlite.ts` - SQLite implementation
- `server/db/migrations/001-multi-tenant-schema.sql` - Schema

### Existing Features
- `server/commands.ts` - Chat command handling
- `server/services/dropgame.ts` - Drop game logic
- `server/services/kick-api.ts` - Kick API integration
- `server/routes/` - API route handlers

### Frontend
- `src/pages/OverlayPage.tsx` - Overlay rendering
- `src/pages/AdminPage.tsx` - Admin dashboard
- `src/components/overlay/` - Widget components

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Streamers onboarded | 100 in first month |
| Daily active streamers | 30% of registered |
| Viewer profiles claimed | 10% of unique chatters |
| Plugin adoption | 50% enable at least one |
| Uptime | 99.9% |
| Support tickets | < 5/week |

---

## Open Questions

1. **Monetization**: Freemium? Which features are paid?
2. **Plugin Marketplace**: Allow third-party plugins?
3. **White Labeling**: Allow streamers to remove branding?
4. **Team Accounts**: Multiple admins per channel?
5. **Analytics Dashboard**: What metrics to show streamers?

---

## Glossary

| Term | Definition |
|------|------------|
| Streamer | Channel owner, primary user |
| Viewer | Chat participant, watches streams |
| Overlay | Browser source displayed on stream |
| Widget | Component within an overlay (chat, goals, etc.) |
| Plugin | Optional feature module |
| Platform | Streaming service (Kick, Twitch) |
| Tenant | Isolated data space (one per streamer) |
| TTS | Text-to-Speech |

---

## Next Steps

1. **Create new project folder**: `streaming-system` or `streamvibes`
2. **Initialize with BMAD**: Run `workflow-init` for greenfield
3. **Phase 0 Discovery**: Brainstorm and research with BMAD team
4. **Phase 1 Planning**: Create PRD from this document
5. **Phase 2 Solutioning**: Architecture and epic planning
6. **Phase 3 Implementation**: Sprint-based development

---

*This document serves as the foundation for BMAD greenfield development.*
*Reference: kick-redirect project for implementation patterns.*
*Ready for BMAD team kickoff!*
