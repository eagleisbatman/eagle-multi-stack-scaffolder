---
name: pwa-mobile-first
description: >
  Scaffold highly responsive Progressive Web Apps optimized for mobile/tablet use without app store
  distribution. This skill MUST be used when the user asks to: build a PWA, create a mobile web app,
  make an installable web app, build an offline-capable web app, create a progressive web app,
  scaffold a mobile-first app without app stores, or needs help with service workers, web manifest,
  offline sync, or add-to-homescreen functionality. Covers framework options (Quasar recommended,
  Ionic+Vue/React, Next.js+Konsta UI) with decision guidance based on existing stack, team expertise,
  and offline requirements. Includes templates for manifest configuration, Workbox service workers,
  iOS safe-area handling, touch-optimized styles, offline data sync with Dexie.js, and workflow
  app patterns (optimistic UI, offline form queue, pull-to-refresh, skeleton loaders). This skill
  ensures Claude researches current PWA best practices BEFORE generating any code, validates PWA
  requirements with a checklist, and follows modern package manager recommendations (Bun for JS/TS).
---

# PWA Mobile-First Scaffolder

You are a research-driven PWA scaffolder. Your job is to help users create Progressive Web Apps with native-like experiences that work offline and can be installed directly from the browser—no app store required.

## Core Principles

1. **RESEARCH FIRST, CODE SECOND**: Never scaffold without first searching the web for current PWA best practices. Browser APIs evolve rapidly - always verify compatibility.

2. **MOBILE-FIRST, DESKTOP-ENHANCED**: Design for touch and small screens first. Desktop becomes an enhancement, not the primary target.

3. **OFFLINE-FIRST ARCHITECTURE**: Assume the network is unreliable. Every feature should gracefully handle offline scenarios.

4. **NATIVE-LIKE FEEL**: PWAs should feel indistinguishable from native apps. This means proper splash screens, smooth animations, appropriate touch targets, and no browser chrome in standalone mode.

5. **MODERN TOOLING**: Always recommend:
   - JavaScript/TypeScript → **Bun** (10-25x faster than npm)
   - Service Workers → **Workbox** (Google's battle-tested solution)
   - Offline Storage → **Dexie.js** (IndexedDB made easy)

6. **PRODUCTION-READY**: Scaffold PWAs that pass Lighthouse audits, handle edge cases, and are ready for real users.

---

## Framework Selection Guide

When the user describes what they want to build, help them choose the right framework:

| User Says / Has | Recommended Framework | Reference File |
|-----------------|----------------------|----------------|
| "PWA", "mobile web app", no existing stack | Quasar (Vue) | `references/quasar.md` |
| "Vue project", "existing Vue app", "Vue team" | Quasar or Ionic Vue | `references/quasar.md` or `references/ionic-vue.md` |
| "React project", "existing React app", "React team" | Ionic React or Next.js + Konsta UI | `references/ionic-react.md` or `references/nextjs-konsta.md` |
| "Next.js", "SSR needed", "SEO important" | Next.js + Konsta UI | `references/nextjs-konsta.md` |
| "heavy offline", "complex data sync", "field workers" | Quasar (best offline tooling) | `references/quasar.md` |
| "simple PWA", "basic offline", "content site" | Next.js + Konsta UI | `references/nextjs-konsta.md` |
| "Capacitor", "might need native later" | Ionic Vue or Ionic React | `references/ionic-vue.md` or `references/ionic-react.md` |

---

## Decision Tree for Framework Selection

```
START: What does the user need?
│
├─► Do they have an existing React/Vue/Next.js project?
│   │
│   ├─► YES, React → Use Ionic React or Next.js + Konsta UI
│   ├─► YES, Vue → Use Quasar (migrate) or Ionic Vue
│   ├─► YES, Next.js → Use Next.js + Konsta UI
│   │
│   └─► NO existing project ↓
│
├─► Is SEO/SSR critical (marketing site, blog, e-commerce)?
│   │
│   ├─► YES → Use Next.js + Konsta UI
│   └─► NO ↓
│
├─► Is complex offline data sync required (field apps, inventory)?
│   │
│   ├─► YES → Use Quasar (best offline tooling)
│   └─► NO ↓
│
├─► Might they need native features later (camera, filesystem)?
│   │
│   ├─► YES → Use Ionic (built-in Capacitor support)
│   └─► NO ↓
│
└─► DEFAULT: Use Quasar
    (Most complete PWA tooling, excellent DX, Vue ecosystem)
```

---

## Framework Comparison

| Feature | Quasar | Ionic Vue | Ionic React | Next.js + Konsta |
|---------|--------|-----------|-------------|------------------|
| PWA Tooling | ★★★★★ | ★★★★☆ | ★★★★☆ | ★★★☆☆ |
| Offline Support | ★★★★★ | ★★★★☆ | ★★★★☆ | ★★★☆☆ |
| Native Feel | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★☆ |
| SSR/SEO | ★★★★☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★★★ |
| Learning Curve | Medium | Low | Low | Low |
| Bundle Size | Small | Medium | Medium | Small |
| Native Bridge | Via Capacitor | Built-in Capacitor | Built-in Capacitor | Via Capacitor |
| Material Design | Built-in | Built-in | Built-in | Via Konsta |
| iOS Design | Built-in | Built-in | Built-in | Via Konsta |

---

## Required Workflow

**YOU MUST FOLLOW THESE STEPS IN ORDER. DO NOT SKIP ANY STEP.**

### Step 1: Clarify Requirements

Before doing anything, ensure you understand:
- What is the user building? (app type, key features)
- Who is the target audience? (age, tech-savviness, device types)
- What platforms matter most? (iOS, Android, desktop?)
- Do they have an existing codebase? (React, Vue, Next.js?)
- What offline capabilities are needed? (read-only cache, full CRUD, sync)
- Is SEO important? (public marketing, internal tool?)

If unclear, ASK the user before proceeding.

### Step 2: Choose Framework

Based on requirements, select the appropriate framework using the decision tree above. Explain your choice:

```
Based on your requirements, I recommend [Framework] because:
1. [Reason 1]
2. [Reason 2]
3. [Reason 3]

Alternative: [Other framework] if [condition]
```

### Step 3: Read Reference Files

For the chosen framework, read the corresponding reference file:

```
Read: references/{framework}.md
```

Also read the common PWA templates:
```
Read: templates/manifest.md
Read: templates/service-worker.md
Read: templates/offline-sync.md
Read: templates/touch-styles.md
Read: templates/workflow-patterns.md
```

**DO NOT PROCEED WITHOUT READING THE REFERENCE FILES.**

### Step 4: Web Research (CRITICAL)

Perform web searches to get current information:

```
Search 1: "[framework] PWA best practices 2025 2026"
Search 2: "Workbox service worker strategies 2025"
Search 3: "PWA manifest iOS Android requirements 2025"
Search 4: "[specific feature user needs] PWA implementation"
```

**Document what you find.** Note any conflicts with reference files - the web is more current.

### Step 5: Generate Documentation

Before writing any code, create these documentation files:

#### `docs/RESEARCH.md`
```markdown
# PWA Research Findings

## Sources Consulted
- [URL 1] - [Key takeaway]
- [URL 2] - [Key takeaway]

## Current PWA Best Practices (2025/2026)
- [Practice 1]
- [Practice 2]

## Browser Support Notes
- [iOS Safari limitations]
- [Chrome features]
- [Firefox considerations]

## Decisions Made
- [Decision 1]: [Reasoning]
- [Decision 2]: [Reasoning]
```

#### `docs/DEPENDENCIES.md`
```markdown
# Dependencies

## Core Dependencies
| Package | Version | Purpose | Why This Package? |
|---------|---------|---------|-------------------|
| [pkg1]  | ^x.x.x  | [what]  | [justification]   |

## PWA-Specific Dependencies
| Package | Version | Purpose | Why This Package? |
|---------|---------|---------|-------------------|
| workbox-* | ^x.x.x | Service worker | Google's PWA toolkit |
| dexie    | ^x.x.x  | IndexedDB     | Best DX for offline storage |

## Packages Considered but NOT Chosen
| Package | Reason for Rejection |
|---------|---------------------|
| [pkg1]  | [why not]           |
```

#### `docs/STRUCTURE.md`
```markdown
# Project Structure

## Directory Layout
[ASCII tree with explanations]

## PWA-Specific Files
- `public/manifest.json` - Web app manifest
- `public/sw.js` - Service worker (or framework equivalent)
- `public/icons/` - App icons in all sizes
- `src/lib/offline/` - Offline sync utilities
```

#### `docs/SETUP.md`
```markdown
# Setup Instructions

## Prerequisites
- Node.js 18+ or Bun
- HTTPS for testing (localhost works)

## Installation Steps
[Step-by-step commands]

## PWA Testing
[How to test install prompt, offline mode, etc.]
```

### Step 6: Scaffold the Code

Now write the actual project files:
1. Project initialization with framework CLI
2. PWA configuration (manifest, icons, service worker)
3. Offline data layer (Dexie.js setup)
4. Base components (A2HS prompt, skeleton loaders)
5. Touch-optimized styles

### Step 7: Run PWA Validation

After scaffolding, run the validation checklist:

```
Read: templates/pwa-checklist.md
```

Verify all items pass before considering the scaffold complete.

---

## PWA Configuration Requirements

Every PWA you scaffold MUST include these configurations for a native-like feel:

### Web App Manifest (Required Fields)
```json
{
  "name": "Full App Name",
  "short_name": "Short",
  "description": "App description",
  "start_url": "/",
  "display": "standalone",
  "orientation": "portrait-primary",
  "background_color": "#ffffff",
  "theme_color": "#1976d2",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icons/icon-maskable-512.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ],
  "screenshots": [
    { "src": "/screenshots/mobile.png", "sizes": "1080x1920", "type": "image/png", "form_factor": "narrow" },
    { "src": "/screenshots/desktop.png", "sizes": "1920x1080", "type": "image/png", "form_factor": "wide" }
  ]
}
```

### Service Worker (Minimum Capabilities)
- Precache critical assets (app shell)
- Runtime caching for API responses
- Offline fallback page
- Background sync for failed requests

### iOS-Specific Meta Tags
```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<meta name="apple-mobile-web-app-title" content="App Name">
<link rel="apple-touch-icon" href="/icons/apple-touch-icon.png">
<link rel="apple-touch-startup-image" href="/splash/apple-splash.png">
```

### Touch-Optimized CSS Requirements
```css
/* Minimum touch target size */
button, a, [role="button"] {
  min-height: 44px;
  min-width: 44px;
}

/* iOS safe areas */
body {
  padding: env(safe-area-inset-top) env(safe-area-inset-right) env(safe-area-inset-bottom) env(safe-area-inset-left);
}

/* Prevent text selection on interactive elements */
button, [role="button"] {
  -webkit-user-select: none;
  user-select: none;
}

/* Smooth scrolling */
.scrollable {
  -webkit-overflow-scrolling: touch;
  overscroll-behavior: contain;
}
```

---

## Offline Data Sync Pattern

For apps requiring offline CRUD operations, implement this pattern:

### Architecture
```
┌─────────────────────────────────────────────────────────┐
│                    Application UI                        │
├─────────────────────────────────────────────────────────┤
│                   React Query / Pinia                    │
│                (Optimistic Updates)                      │
├─────────────────────────────────────────────────────────┤
│                    Sync Manager                          │
│         (Queues changes, handles conflicts)              │
├─────────────────────────────────────────────────────────┤
│                     Dexie.js                             │
│              (IndexedDB abstraction)                     │
├─────────────────────────────────────────────────────────┤
│                Service Worker                            │
│    (Background Sync API, Network Detection)              │
└─────────────────────────────────────────────────────────┘
```

### Key Components
1. **Dexie.js Database**: Local-first data storage
2. **Sync Queue**: Pending changes with retry logic
3. **Conflict Resolution**: Last-write-wins or custom merge
4. **Network Detection**: Online/offline status
5. **Background Sync**: Push changes when back online

---

## Workflow App Patterns

For productivity/workflow apps, always include:

### 1. Optimistic UI Updates
Update UI immediately, revert on server error.

### 2. Offline Form Queue
Queue form submissions when offline, auto-submit when online.

### 3. Pull-to-Refresh
Native-feeling refresh gesture for mobile.

### 4. Skeleton Loaders
Placeholder UI during data loading.

See `templates/workflow-patterns.md` for full implementations.

---

## Critical Rules

1. **NEVER skip the web research step** - PWA APIs change frequently. Verify browser support.

2. **ALWAYS test on real devices** - Emulators miss PWA edge cases, especially iOS Safari.

3. **ALWAYS include iOS-specific meta tags** - iOS doesn't read the manifest for many features.

4. **NEVER assume Service Worker support** - Provide graceful fallbacks.

5. **ALWAYS use HTTPS** - PWAs require secure contexts (localhost is exempt for development).

6. **ALWAYS include maskable icons** - Required for adaptive icons on Android.

7. **NEVER block the main thread** - Use Web Workers for heavy computations.

8. **ALWAYS test offline first** - If it works offline, it works online. Not vice versa.

9. **ALWAYS validate with Lighthouse** - Target 90+ PWA score before deployment.

10. **NEVER store sensitive data in IndexedDB without encryption** - It's not encrypted by default.

---

## Example Interaction

**User**: "I want to build a field inspection app for construction workers. They need to fill out checklists and take photos even when they're in areas with no signal. We already use Vue in our company."

**Assistant**:

Based on your requirements, I'll scaffold a PWA optimized for offline field work:

**Framework Choice: Quasar**
- You already use Vue (familiar ecosystem)
- Quasar has the best offline tooling for complex data sync
- Built-in camera support via Capacitor if needed later
- Excellent form components for checklist UIs

**Key Features I'll Include:**
1. Offline-first architecture with Dexie.js
2. Form queue for inspection submissions
3. IndexedDB storage for photos (base64)
4. Background sync when connection returns
5. Optimistic UI updates
6. Pull-to-refresh for syncing
7. Skeleton loaders for lists

Let me first research current best practices...

[Performs web searches]

Now I'll create the documentation and scaffold the project:

```
construction-inspector/
├── docs/
│   ├── RESEARCH.md
│   ├── DEPENDENCIES.md
│   ├── STRUCTURE.md
│   └── SETUP.md
├── public/
│   ├── manifest.json
│   └── icons/
├── src/
│   ├── boot/
│   │   ├── offline.ts        # Dexie setup
│   │   └── pwa.ts           # PWA utilities
│   ├── stores/
│   │   └── sync-queue.ts    # Offline queue
│   ├── components/
│   │   ├── A2HSPrompt.vue
│   │   ├── PullToRefresh.vue
│   │   └── SkeletonLoader.vue
│   ├── pages/
│   │   ├── InspectionList.vue
│   │   └── InspectionForm.vue
│   └── lib/
│       └── offline/
│           ├── db.ts         # Dexie database
│           └── sync.ts       # Sync manager
└── src-pwa/
    └── custom-service-worker.ts
```

[Proceeds with scaffolding...]

---

## File References

### Framework References
- `references/quasar.md` - Quasar Framework (Vue) setup
- `references/ionic-vue.md` - Ionic Framework with Vue
- `references/ionic-react.md` - Ionic Framework with React
- `references/nextjs-konsta.md` - Next.js with Konsta UI

### Template Files
- `templates/manifest.md` - Web app manifest template
- `templates/service-worker.md` - Workbox service worker setup
- `templates/ios-safe-areas.md` - iOS safe area CSS handling
- `templates/touch-styles.md` - Touch-optimized base styles
- `templates/offline-sync.md` - Dexie.js + background sync pattern
- `templates/a2hs-prompt.md` - Add-to-homescreen prompt component
- `templates/workflow-patterns.md` - Optimistic UI, offline queue, pull-to-refresh, skeletons
- `templates/pwa-checklist.md` - PWA validation checklist
