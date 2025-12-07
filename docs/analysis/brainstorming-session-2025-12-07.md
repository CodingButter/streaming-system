---
stepsCompleted: [1]
inputDocuments: ['STREAMING-SYSTEM-DESIGN.md']
session_topic: 'StreamVibes - Multi-platform streaming overlay SaaS'
session_goals: 'Feature prioritization, competitive differentiation, architecture validation, blind spot discovery'
selected_approach: 'AI-Recommended Techniques'
techniques_used: []
ideas_generated: []
context_file: 'STREAMING-SYSTEM-DESIGN.md'
---

# Brainstorming Session Results

**Facilitator:** Butters
**Date:** 2025-12-07

## Session Overview

**Topic:** StreamVibes - Multi-platform streaming overlay SaaS for Kick and Twitch streamers

**Goals:**
- Validate MVP feature set and prioritization
- Identify killer features that differentiate from StreamElements/Streamlabs
- Explore plugin ecosystem potential
- Surface risks and blind spots in the current design

### Context Guidance

Based on `STREAMING-SYSTEM-DESIGN.md`:
- Multi-tenant SaaS with plugin architecture
- Core features: overlays, chat, events, goals, tips, commands, viewer profiles
- Plugins: Drop Game, AI Chatbot, future expansions
- Target: Small-medium streamers (10-1000 viewers), simulcasters
- Tech: Bun, SQLite, React, WebSocket

### Session Setup

**Approach Selected:** AI-Recommended Techniques

## Technique Selection

**Approach:** AI-Recommended Techniques
**Analysis Context:** StreamVibes multi-platform SaaS with focus on differentiation, validation, and blind spots

**Recommended Techniques:**

1. **Assumption Reversal** (deep) - Challenge core design assumptions to surface blind spots
2. **Cross-Pollination** (creative) - Transfer winning patterns from other industries for killer features
3. **Chaos Engineering** (wild) - Stress-test the design to find failure modes

**AI Rationale:** Given the comprehensive design document, the session benefits from challenging existing assumptions before adding features. Cross-pollination ensures differentiation beyond streaming industry norms. Chaos engineering validates architecture resilience.

---

## Phase 1: Assumption Reversal - Insights

### Validated Assumptions
| Assumption | Decision | Rationale |
|------------|----------|-----------|
| Single shared database | ✅ Keep | Simpler migrations, cross-streamer analytics possible |
| Plugin opt-in store | ✅ Keep | Lean, no wasted processing on unused features |
| Viewer profiles per-platform | ✅ Keep | `!profile` shows correct link per platform |

### Updated Assumptions
| Original | Updated | Rationale |
|----------|---------|-----------|
| Kick-first, Twitch later | **Both platforms from day one** | Platform adapters, true multi-platform differentiator |
| Separate platform management | **Unified admin, configurable widgets** | One dashboard; widgets can be platform-specific OR merged |

### Key Architecture Insight
> **Streamers manage both platforms as ONE account.** Widgets can be:
> - Platform-specific (Kick goals vs Twitch goals)
> - Merged/unified (combined goal showing both platforms)
>
> Viewers stay per-platform with separate profiles, but the streamer experience is unified.

---

## Phase 2: Cross-Pollination - Insights

### Reference Implementation: kick-redirect
Explored `/home/codingbutter/kick-redirect` - a production-ready single-tenant implementation with:

**Already Built Features (to port):**
- Drop Game with powerups (TNT, Shield, Magnet, Ghost, Boost)
- Gambling mechanics (!roll, !coinflip, !duel)
- Points economy (watch, chat, subs, gifts, tips)
- TTS with 30+ ElevenLabs voices
- AI Chatbot integration
- Overlay system (chat, goals, tips, events widgets)
- Layout editor (grid-based drag-and-drop)
- Theme system (colors, fonts)
- 50+ chat commands
- User profiles with verification
- Admin dashboard

### Critical Platform Shift
| kick-redirect | StreamVibes |
|---------------|-------------|
| Self-hosted, clone repo | Fully hosted SaaS (like Streamlabs) |
| Single streamer per instance | Multi-tenant: many streamers, one platform |
| Kick only | Kick + Twitch with adapters |
| localhost URLs | Public URLs: `streamvibes.com/:platform/:channel/*` |

### StreamVibes Delivery Model
- **Overlay URLs** - Streamers paste into OBS
- **Alert Endpoints** - Follow/sub/tip popups
- **Smart Bot** - StreamVibes bot joins their channel, handles commands
- **Admin Dashboard** - Configure everything in browser
- **No cloning, no self-hosting** - Just sign up and connect

---

## Phase 3: Chaos Engineering - Considerations

### Identified Failure Scenarios
1. **Scale Bomb** - 50K viewers, 5K `!drop` commands in 10 seconds
2. **Platform API Outage** - Kick/Twitch down mid-stream
3. **Multi-Tenant Security** - Data leak between streamers
4. **Bot Spam Attack** - Rate limiting, anomaly detection

### Mitigation Strategies (for Architecture phase)
- Rate limiting per user, per command, per streamer
- Row-level security with `streamer_id` on ALL queries
- Graceful degradation when platform APIs fail
- Queue-based command processing for scale

---

## Additional Features Discovered

### AI Chatbot Plugin (Enhanced)
Streamers can configure:
- **LLM Provider**: OpenAI (ChatGPT), Anthropic (Claude), Google (Gemini)
- **API Key**: Streamer provides their own key
- **System Prompt**: Custom personality/behavior
- **Trigger Configuration**: When bot responds vs stays silent

This makes the AI Chatbot a true plugin where streamers control costs and personality.

---

## Summary: StreamVibes Core Differentiators

1. **Fully Hosted Multi-Platform SaaS** - No self-hosting, supports Kick + Twitch from day one
2. **Visual Layout Builder** - Drag-and-drop overlay creation in admin dashboard
3. **Drop Game** - Unique interactive viewer game with powerup shop
4. **Gambling Mechanics** - !roll, !coinflip, !duel driving engagement and promoting subs/donations
5. **Configurable AI Chatbot** - Choose LLM provider, bring your own API key
6. **Unified Admin** - One dashboard for both platforms, configurable merged or separate widgets
7. **Plugin Architecture** - Opt-in features keep it lean

---

## Next Steps

Brainstorming complete! Ready for **PRD creation** with the PM agent.
