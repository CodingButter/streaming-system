---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
inputDocuments: ['STREAMING-SYSTEM-DESIGN.md', 'docs/analysis/brainstorming-session-2025-12-07.md', '/home/codingbutter/kick-redirect (reference implementation)']
workflowType: 'prd'
lastStep: 11
project_name: 'StreamVibes'
user_name: 'Butters'
date: '2025-12-07'
---

# Product Requirements Document - StreamVibes

**Author:** Butters
**Date:** 2025-12-07

## Executive Summary

**StreamVibes** is a fully hosted, multi-tenant SaaS platform providing customizable stream overlays, chat bot integration, and viewer engagement tools for content creators on Kick and Twitch. Unlike competitors that bolt on platform support as an afterthought, StreamVibes supports both platforms from day one with a unified admin experience.

### What Makes This Special

1. **True Multi-Platform** - Kick and Twitch with platform adapters, not afterthoughts
2. **Drop Game & Gambling** - Unique viewer engagement mechanics that drive subs and donations
3. **Configurable AI Chatbot** - Streamers choose their LLM (OpenAI, Anthropic, Gemini) and bring their own API key
4. **Visual Layout Builder** - Drag-and-drop overlay creation in the browser
5. **Plugin Architecture** - Opt-in features keep it lean and fast

## Project Classification

**Technical Type:** SaaS B2B Platform
**Domain:** Entertainment/Streaming (General)
**Complexity:** Medium

Multi-tenant architecture with real-time WebSocket connections, platform API integrations, but no heavy regulatory compliance requirements.

## Success Criteria

### User Success

A streamer can:
- Sign up with Kick or Twitch OAuth
- Build a custom overlay layout with drag-and-drop
- Add core widgets (chat, events, goals, tips)
- Copy overlay URL and use it in OBS
- See real-time updates on their overlay

### Technical Success

- Overlay updates < 100ms latency
- Multi-tenant isolation verified (no data leaks)
- Platform adapters work for both Kick and Twitch
- WebSocket connections stable during streams
- Professional SaaS quality (uptime, error handling)

### Measurable Outcomes

- All MVP features functional end-to-end
- Streamer onboarding flow works without manual intervention
- Overlay renders correctly across browsers

## Product Scope

### MVP - Minimum Viable Product

- Streamer OAuth (Kick + Twitch)
- Admin dashboard
- Overlay layout builder (drag-and-drop)
- Core widgets: Chat, Events, Goals, Tips
- Theme customization
- Public overlay URLs for OBS
- Multi-tenant data isolation

### Post-MVP (Plugins)

- Drop Game
- AI Chatbot (LLM selection)
- Gambling commands (!roll, !coinflip, !duel)
- Points economy
- TTS integration
- Polls & Predictions

### Vision (Future)

- Third-party plugin marketplace
- Team accounts
- Analytics dashboard
- Community theme sharing

## User Journeys

### Journey 1: Alex Chen - First-Time Streamer Setup

Alex is a growing Kick streamer with about 200 regular viewers. He's been using basic OBS scenes but wants professional overlays without paying enterprise prices or fighting with complicated tools. He finds StreamVibes and clicks "Get Started."

After connecting his Kick account with one click, Alex lands in his admin dashboard. He opens the layout builder and drags a chat widget to the right side, an events widget to the top for follow alerts, and a goals widget showing his subscriber progress. He picks a dark theme that matches his brand colors.

Alex copies his overlay URL, pastes it into OBS as a browser source, and hits "Start Streaming." His chat lights up, a follower notification pops on screen, and his goal bar updates in real-time. Setup took 10 minutes instead of the hours he expected.

### Journey 2: Adding a Second Platform

A month later, Alex starts simulcasting to Twitch. He goes to Settings, clicks "Connect Twitch," and authorizes. Now his dashboard shows both platforms. He creates a second overlay URL for Twitch, or chooses to merge both chats into one combined widget. His viewers on both platforms see the same professional experience.

### Journey Requirements Summary

These journeys reveal MVP requirements for:
- OAuth signup (Kick + Twitch)
- Admin dashboard with layout builder
- Core widgets (chat, events, goals)
- Theme customization
- Public overlay URLs
- Multi-platform connection flow

## SaaS Platform Requirements

### Multi-Tenant Architecture

- Row-level isolation with `streamer_id` on all tables
- No streamer can access another streamer's data
- Foreign keys cascade deletes for clean data removal

### Permission Model

| Role | Access |
|------|--------|
| Streamer | Full access to own data, admin dashboard |
| Viewer | Per-channel profile, can customize voice/avatar |

### Integrations (MVP)

| Service | Purpose |
|---------|---------|
| Kick API | OAuth, chat, webhooks, goals |
| Twitch API | OAuth, EventSub, chat (IRC pattern) |

### Not in MVP

- Payment/subscription tiers
- Analytics dashboard
- Third-party integrations beyond platform APIs

## Functional Requirements

### Authentication & Account Management

- FR1: Streamer can sign up using Kick OAuth
- FR2: Streamer can sign up using Twitch OAuth
- FR3: Streamer can connect additional platform after initial signup
- FR4: Streamer can disconnect a platform from their account
- FR5: Streamer can view all connected platforms in dashboard

### Overlay Layout Management

- FR6: Streamer can create a new overlay layout
- FR7: Streamer can drag-and-drop widgets onto layout grid
- FR8: Streamer can reposition widgets within the layout
- FR9: Streamer can remove widgets from layout
- FR10: Streamer can save layout configuration
- FR11: Streamer can create multiple saved layouts
- FR12: Streamer can set a default layout
- FR13: Streamer can duplicate an existing layout
- FR14: Streamer can delete a layout

### Core Widgets

- FR15: Streamer can add Chat widget to layout
- FR16: Streamer can add Events widget to layout (follows, subs, tips)
- FR17: Streamer can add Goals widget to layout (follower/sub progress)
- FR18: Streamer can add Tips widget to layout (rotating messages)
- FR19: Streamer can configure individual widget settings

### Theme Customization

- FR20: Streamer can customize global color scheme
- FR21: Streamer can customize fonts
- FR22: Streamer can override colors per widget
- FR23: Streamer can save theme configuration
- FR24: Streamer can export/import theme

### Public Overlay Access

- FR25: System provides unique overlay URL per streamer per platform
- FR26: Overlay URL renders layout in real-time for OBS browser source
- FR27: Overlay receives real-time chat messages via WebSocket
- FR28: Overlay receives real-time events (follows, subs, tips) via WebSocket
- FR29: Overlay updates goals in real-time from platform API

### Platform Integration

- FR30: System connects to Kick chat via WebSocket
- FR31: System connects to Twitch chat via IRC pattern
- FR32: System receives Kick webhooks for events
- FR33: System receives Twitch EventSub for events
- FR34: System fetches follower/subscriber counts from platform APIs

### Admin Dashboard

- FR35: Streamer can access admin dashboard after authentication
- FR36: Streamer can navigate between dashboard sections
- FR37: Streamer can view overlay preview in dashboard

### Data Isolation

- FR38: System isolates all streamer data by streamer_id
- FR39: Streamer cannot access another streamer's data
- FR40: System cascades deletes when streamer account removed

## Non-Functional Requirements

### Performance

- NFR1: Overlay updates render within 100ms of event receipt
- NFR2: Admin dashboard pages load within 2 seconds
- NFR3: WebSocket message delivery latency < 50ms
- NFR4: Database queries complete within 50ms
- NFR5: System supports 1000 concurrent viewers per streamer

### Security

- NFR6: OAuth tokens encrypted at rest
- NFR7: All API endpoints require authentication (except public overlay URLs)
- NFR8: Row-level data isolation enforced on every query
- NFR9: Webhook signatures verified before processing
- NFR10: HTTPS required for all connections
- NFR11: No SQL injection vulnerabilities (prepared statements only)
- NFR12: XSS prevention via React escaping and CSP headers

### Reliability

- NFR13: WebSocket connections auto-reconnect on disconnect
- NFR14: Graceful degradation when platform APIs unavailable
- NFR15: Webhook retry with exponential backoff on failure
- NFR16: System recovers from crashes without data loss
- NFR17: 99.9% uptime target

### Integration

- NFR18: Platform adapters support Kick and Twitch APIs
- NFR19: System handles platform API rate limits gracefully
- NFR20: OAuth token refresh happens automatically before expiry
