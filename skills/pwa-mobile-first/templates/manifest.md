# Web App Manifest Template

## Overview

The web app manifest is a JSON file that tells the browser about your PWA and how it should behave when installed. A complete manifest is required for the "Add to Home Screen" prompt.

## Complete Manifest Template

```json
{
  "name": "Your App Full Name",
  "short_name": "AppName",
  "description": "A brief description of your app (max 300 chars)",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait-primary",
  "background_color": "#ffffff",
  "theme_color": "#1976d2",
  "dir": "ltr",
  "lang": "en-US",
  "categories": ["productivity", "utilities"],
  "iarc_rating_id": "",
  "prefer_related_applications": false,
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-maskable-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/mobile-home.png",
      "sizes": "1080x1920",
      "type": "image/png",
      "form_factor": "narrow",
      "label": "Home screen on mobile"
    },
    {
      "src": "/screenshots/mobile-feature.png",
      "sizes": "1080x1920",
      "type": "image/png",
      "form_factor": "narrow",
      "label": "Main feature on mobile"
    },
    {
      "src": "/screenshots/desktop-home.png",
      "sizes": "1920x1080",
      "type": "image/png",
      "form_factor": "wide",
      "label": "Home screen on desktop"
    }
  ],
  "shortcuts": [
    {
      "name": "New Item",
      "short_name": "New",
      "description": "Create a new item",
      "url": "/new?source=shortcut",
      "icons": [
        {
          "src": "/icons/shortcut-new.png",
          "sizes": "192x192",
          "type": "image/png"
        }
      ]
    },
    {
      "name": "Search",
      "short_name": "Search",
      "description": "Search items",
      "url": "/search?source=shortcut",
      "icons": [
        {
          "src": "/icons/shortcut-search.png",
          "sizes": "192x192",
          "type": "image/png"
        }
      ]
    }
  ],
  "share_target": {
    "action": "/share",
    "method": "POST",
    "enctype": "multipart/form-data",
    "params": {
      "title": "title",
      "text": "text",
      "url": "url",
      "files": [
        {
          "name": "media",
          "accept": ["image/*", "video/*"]
        }
      ]
    }
  },
  "protocol_handlers": [
    {
      "protocol": "web+myapp",
      "url": "/handle?url=%s"
    }
  ],
  "related_applications": [],
  "edge_side_panel": {
    "preferred_width": 400
  }
}
```

## Required Fields

These fields are **required** for PWA installability:

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Full application name (max 45 chars recommended) | "Task Manager Pro" |
| `short_name` | Shown on home screen (max 12 chars) | "Tasks" |
| `start_url` | URL when app launches | "/" or "/app" |
| `display` | Display mode | "standalone" |
| `icons` | At least 192x192 and 512x512 | See template |

## Display Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `standalone` | App-like, no browser UI | Most PWAs |
| `fullscreen` | No system UI at all | Games, immersive apps |
| `minimal-ui` | Minimal browser controls | Apps needing back button |
| `browser` | Normal browser tab | Not recommended for PWAs |

**Recommendation**: Use `standalone` for most apps.

## Icon Requirements

### Minimum Required Icons
- **192x192** - Android install prompt
- **512x512** - Splash screen, app stores

### Recommended Icon Sizes
```
72x72    - Android (legacy)
96x96    - Android (legacy)
128x128  - Chrome Web Store
144x144  - Windows tiles
152x152  - iOS (iPad)
192x192  - Android (required)
384x384  - Android (high-res)
512x512  - Splash screen (required)
```

### Maskable Icons

Maskable icons are **required** for Android adaptive icons:

```json
{
  "src": "/icons/icon-maskable-512x512.png",
  "sizes": "512x512",
  "type": "image/png",
  "purpose": "maskable"
}
```

**Design Guidelines for Maskable Icons:**
- Keep important content in the center 80% "safe zone"
- Background should extend to all edges
- Use https://maskable.app to preview

### Icon Generation Tools
- [RealFaviconGenerator](https://realfavicongenerator.net) - Best overall
- [PWA Asset Generator](https://github.com/nicholasbalka/pwa-asset-generator) - CLI tool
- [Maskable.app](https://maskable.app) - Preview maskable icons

## Screenshots

Screenshots trigger the "richer install UI" on Android:

```json
"screenshots": [
  {
    "src": "/screenshots/mobile-1.png",
    "sizes": "1080x1920",
    "type": "image/png",
    "form_factor": "narrow",
    "label": "Home screen"
  },
  {
    "src": "/screenshots/desktop-1.png",
    "sizes": "1920x1080",
    "type": "image/png",
    "form_factor": "wide",
    "label": "Dashboard view"
  }
]
```

**Requirements:**
- At least 2 screenshots recommended
- `form_factor`: "narrow" for mobile, "wide" for desktop
- Aspect ratio between 0.5 and 2.0
- Same aspect ratio for all screenshots of same form_factor

## Shortcuts (App Shortcuts)

Quick actions from long-press on app icon:

```json
"shortcuts": [
  {
    "name": "Create New",
    "short_name": "New",
    "description": "Create a new item",
    "url": "/new?source=shortcut",
    "icons": [{ "src": "/icons/shortcut-new.png", "sizes": "192x192" }]
  }
]
```

**Limits:**
- Maximum 4 shortcuts recommended
- Icons should be 192x192 minimum
- Use query params to track shortcut usage

## Share Target

Allow your PWA to receive shared content:

```json
"share_target": {
  "action": "/share",
  "method": "POST",
  "enctype": "multipart/form-data",
  "params": {
    "title": "title",
    "text": "text",
    "url": "url",
    "files": [
      {
        "name": "media",
        "accept": ["image/*", "video/*"]
      }
    ]
  }
}
```

**Handler Example (Next.js):**
```typescript
// app/share/route.ts
export async function POST(request: Request) {
  const formData = await request.formData();
  const title = formData.get('title');
  const text = formData.get('text');
  const url = formData.get('url');
  const files = formData.getAll('media');

  // Process shared content
  // Redirect to app with data
  return Response.redirect('/app?shared=true');
}
```

## iOS Considerations

iOS Safari **ignores most manifest fields**. You need these meta tags in HTML:

```html
<!-- iOS-specific meta tags -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<meta name="apple-mobile-web-app-title" content="App Name">

<!-- iOS icons -->
<link rel="apple-touch-icon" href="/icons/apple-touch-icon-180x180.png">

<!-- iOS splash screens -->
<link rel="apple-touch-startup-image"
      href="/splash/apple-splash-2048-2732.jpg"
      media="(device-width: 1024px) and (device-height: 1366px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)">
<!-- Add more for all device sizes -->
```

### iOS Status Bar Styles

| Value | Description |
|-------|-------------|
| `default` | White bar, black text |
| `black` | Black bar, white text |
| `black-translucent` | Transparent, content extends under |

**Recommendation**: Use `default` for light themes, `black-translucent` for immersive apps.

## HTML Head Template

```html
<head>
  <!-- Primary Meta -->
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">

  <!-- PWA Manifest -->
  <link rel="manifest" href="/manifest.json">

  <!-- Theme Color -->
  <meta name="theme-color" content="#1976d2">

  <!-- iOS PWA Meta -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="default">
  <meta name="apple-mobile-web-app-title" content="App Name">

  <!-- Icons -->
  <link rel="icon" type="image/png" sizes="32x32" href="/icons/favicon-32x32.png">
  <link rel="icon" type="image/png" sizes="16x16" href="/icons/favicon-16x16.png">
  <link rel="apple-touch-icon" href="/icons/apple-touch-icon-180x180.png">

  <!-- Microsoft Tiles -->
  <meta name="msapplication-TileColor" content="#1976d2">
  <meta name="msapplication-TileImage" content="/icons/mstile-144x144.png">

  <!-- Description for SEO -->
  <meta name="description" content="Your app description">
</head>
```

## Validation

Use these tools to validate your manifest:

1. **Chrome DevTools** → Application → Manifest
2. **Lighthouse PWA Audit**
3. **PWA Builder** - https://www.pwabuilder.com/
4. **Web.dev PWA Checklist** - https://web.dev/pwa-checklist/

## Common Issues

| Issue | Solution |
|-------|----------|
| Install prompt not showing | Check HTTPS, manifest link, service worker |
| Wrong icon on home screen | Verify icon paths, check iOS apple-touch-icon |
| App opens in browser | Check `display: standalone`, iOS meta tags |
| Theme color not applying | Ensure `theme_color` in manifest AND meta tag |
| Splash screen blank | Check `background_color`, icon sizes |
