---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments: ['docs/prd.md', 'docs/analysis/brainstorming-session-2025-12-07.md']
workflowType: 'ux-design'
lastStep: 0
project_name: 'StreamVibes'
user_name: 'Butters'
date: '2025-12-07'
---

# UX Design Specification - StreamVibes

**Author:** Butters
**Date:** 2025-12-07

---

## Design System Overview

StreamVibes inherits a proven design system from the kick-redirect reference implementation. No significant changes required - porting existing UX to multi-tenant SaaS.

### UI Framework

- **Component Library:** shadcn/ui
- **CSS Framework:** Tailwind CSS
- **Theming:** CSS variables for per-user customization

### Dynamic Theming Architecture

Each streamer's theme is stored in the database and injected at runtime:

```html
<head>
  <style>
    :root {
      --primary: ${streamer.primaryColor};
      --secondary: ${streamer.secondaryColor};
      --accent: ${streamer.accentColor};
      --radius: ${streamer.borderRadius};
      --padding: ${streamer.spacing};
      /* ... additional variables */
    }
  </style>
</head>
```

**Theme Variables:**
- Primary, secondary, accent colors
- Border radius
- Spacing/padding
- Font selections
- Background colors

## Application Structure

### Public Pages

| Page | URL Pattern | Purpose |
|------|-------------|---------|
| Landing | `/` | Marketing, signup CTA |
| Overlay | `/:platform/:channel/overlay` | OBS browser source |
| Chat Widget | `/:platform/:channel/chat` | Standalone chat overlay |
| Events Widget | `/:platform/:channel/events` | Follow/sub/tip alerts |
| Goals Widget | `/:platform/:channel/goals` | Progress bars |
| Viewer Profile | `/:platform/:channel/profile/:username` | Viewer stats & settings |

### Authenticated Pages

| Page | URL Pattern | Purpose |
|------|-------------|---------|
| Dashboard | `/dashboard` | Streamer home |
| Layout Editor | `/dashboard/overlay` | Drag-and-drop builder |
| Theme Editor | `/dashboard/theme` | Color/font customization |
| Settings | `/dashboard/settings` | Account & platform connections |
| Widgets | `/dashboard/widgets` | Widget configuration |

## Core User Flows

### Flow 1: Streamer Signup

1. Land on homepage
2. Click "Get Started" or "Sign Up"
3. Choose platform (Kick or Twitch)
4. OAuth redirect to platform
5. Authorize StreamVibes
6. Redirect to dashboard (first-time setup)

### Flow 2: Build Overlay

1. Navigate to Layout Editor
2. View 12x9 grid (1920x1080 reference)
3. Click "Add Widget" to open picker
4. Drag widget to desired position
5. Click widget to configure settings
6. Save layout
7. Copy overlay URL
8. Paste in OBS as browser source

### Flow 3: Customize Theme

1. Navigate to Theme Editor
2. Use color pickers for primary/secondary/accent
3. Adjust border radius slider
4. Select fonts from dropdown
5. Preview changes in real-time
6. Save theme

### Flow 4: Add Second Platform

1. Navigate to Settings
2. Click "Connect Platform"
3. Select Kick or Twitch
4. OAuth flow
5. Return to dashboard with both platforms shown

## Component Inventory

### Layout Editor Components

- **Grid Canvas:** 12x9 drag-and-drop area
- **Widget Picker:** Modal with available widgets
- **Widget Handle:** Draggable/resizable widget container
- **Widget Settings Panel:** Slide-out configuration
- **Layout Selector:** Dropdown for saved layouts
- **Preview Toggle:** Show/hide live preview

### Dashboard Components

- **Sidebar Navigation:** Main nav menu
- **Platform Switcher:** Toggle between Kick/Twitch views
- **Stat Cards:** Quick metrics display
- **Action Buttons:** Primary CTAs

### Overlay Components (for OBS)

- **Chat Widget:** Scrolling chat messages
- **Events Widget:** Animated alert popups
- **Goals Widget:** Progress bars with labels
- **Tips Widget:** Rotating text messages

## Responsive Behavior

| Breakpoint | Target |
|------------|--------|
| Desktop (1280px+) | Primary - full dashboard experience |
| Tablet (768-1279px) | Simplified layout editor |
| Mobile (< 768px) | Settings & viewing only, no editing |

**Note:** Overlay pages are fixed at 1920x1080 for OBS compatibility.

## Accessibility Considerations

- Dashboard meets WCAG 2.1 AA for admin tasks
- Overlay pages prioritize visual design (not accessibility-critical as they're display-only)
- Keyboard navigation supported in dashboard
- Color contrast maintained in default themes
