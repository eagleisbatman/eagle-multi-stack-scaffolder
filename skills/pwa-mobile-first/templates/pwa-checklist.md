# PWA Validation Checklist

## Overview

Use this checklist to validate that your PWA meets all requirements for installability, offline functionality, and native-like experience. Run through this before considering the scaffold complete.

---

## 1. Installability Requirements

### Web App Manifest
- [ ] `manifest.json` or `manifest.webmanifest` exists
- [ ] Linked in HTML: `<link rel="manifest" href="/manifest.json">`
- [ ] `name` is present and descriptive (max 45 chars recommended)
- [ ] `short_name` is present (max 12 chars)
- [ ] `start_url` is present (usually "/" or "/app")
- [ ] `display` is set to `standalone` or `fullscreen`
- [ ] `background_color` is set
- [ ] `theme_color` is set
- [ ] Icons array includes 192x192 PNG
- [ ] Icons array includes 512x512 PNG
- [ ] At least one icon has `purpose: "maskable"`

### Service Worker
- [ ] Service worker file exists
- [ ] Service worker is registered in the app
- [ ] Service worker activates successfully (check DevTools)
- [ ] `fetch` event handler is implemented

### HTTPS
- [ ] App is served over HTTPS (or localhost for development)
- [ ] No mixed content warnings

---

## 2. Offline Functionality

### Core Offline
- [ ] App shell (HTML, CSS, JS) is cached on install
- [ ] App loads when offline (even if with limited functionality)
- [ ] Offline fallback page exists and is cached
- [ ] Custom offline page is shown for uncached navigation

### Caching Strategies
- [ ] Static assets use CacheFirst strategy
- [ ] API calls use NetworkFirst with cache fallback
- [ ] Images are cached with size/time limits
- [ ] Fonts are cached

### Data Persistence
- [ ] IndexedDB or localStorage is used for local data
- [ ] User-generated data persists across sessions
- [ ] Data syncs when coming back online
- [ ] Sync queue handles failed requests

### Network Detection
- [ ] App detects online/offline status
- [ ] UI indicates when offline
- [ ] Pending changes are shown to user

---

## 3. iOS Safari Compatibility

### Meta Tags
- [ ] `<meta name="apple-mobile-web-app-capable" content="yes">`
- [ ] `<meta name="apple-mobile-web-app-status-bar-style" content="default">`
- [ ] `<meta name="apple-mobile-web-app-title" content="App Name">`
- [ ] `<link rel="apple-touch-icon" href="/icons/apple-touch-icon.png">`

### Safe Areas
- [ ] `viewport-fit=cover` in viewport meta tag
- [ ] `env(safe-area-inset-*)` used for notch handling
- [ ] Fixed headers account for safe-area-inset-top
- [ ] Fixed footers account for safe-area-inset-bottom
- [ ] Landscape orientation handles left/right safe areas

### iOS-Specific Testing
- [ ] Tested on real iOS device (not just simulator)
- [ ] Home screen icon displays correctly
- [ ] Splash screen shows during launch
- [ ] Status bar style is correct
- [ ] App doesn't show address bar in standalone mode

---

## 4. Touch Optimization

### Touch Targets
- [ ] All interactive elements are at least 44x44 CSS pixels
- [ ] Adequate spacing between touch targets (8px minimum)
- [ ] Touch targets don't overlap

### Gestures
- [ ] Pull-to-refresh works on scrollable content
- [ ] Swipe gestures feel native (if implemented)
- [ ] No accidental scrolling during gestures
- [ ] Touch feedback is immediate

### Interaction
- [ ] `-webkit-tap-highlight-color` is customized or disabled
- [ ] Text selection is disabled on buttons/interactive elements
- [ ] No 300ms tap delay (handled by modern browsers)
- [ ] Long-press doesn't interfere with expected behavior

---

## 5. Performance

### Loading
- [ ] First Contentful Paint < 2s on 4G
- [ ] Time to Interactive < 5s on 4G
- [ ] Lighthouse Performance score > 80

### Optimization
- [ ] Images are optimized and properly sized
- [ ] Fonts use `font-display: swap`
- [ ] JavaScript is code-split
- [ ] CSS is minified
- [ ] No render-blocking resources

### Caching
- [ ] App shell loads from cache on repeat visits
- [ ] API responses are cached appropriately
- [ ] Stale-while-revalidate for frequently updated data

---

## 6. User Experience

### Add to Home Screen
- [ ] A2HS prompt is shown to eligible users
- [ ] iOS instructions are shown to iOS users
- [ ] Prompt can be dismissed
- [ ] Dismissed users aren't prompted again for X days
- [ ] Already-installed users don't see prompt

### Updates
- [ ] Update prompt shown when new version available
- [ ] Update can be installed without losing data
- [ ] skipWaiting is handled properly
- [ ] User understands what "update" means

### Offline UX
- [ ] Offline indicator is visible when offline
- [ ] Pending sync count is shown
- [ ] Failed submissions can be retried
- [ ] Error messages are user-friendly

### Loading States
- [ ] Skeleton loaders for content
- [ ] Loading indicators for async operations
- [ ] Optimistic updates for user actions
- [ ] No spinner-only pages

---

## 7. Lighthouse PWA Audit

Run Lighthouse in Chrome DevTools → Lighthouse → PWA category.

### Installable
- [ ] Uses HTTPS
- [ ] Registers a service worker
- [ ] Web app manifest meets criteria

### PWA Optimized
- [ ] Redirects HTTP to HTTPS
- [ ] Configured for custom splash screen
- [ ] Sets address bar theme color
- [ ] Content is sized correctly for viewport
- [ ] Has `<meta name="viewport">` tag
- [ ] Provides valid apple-touch-icon
- [ ] Maskable icon is provided

**Target**: 90+ PWA score

---

## 8. Validation Script

Run this script to validate key PWA requirements:

```javascript
// pwa-validator.js
// Run in browser console or as a build step

async function validatePWA() {
  const results = {
    passed: [],
    failed: [],
    warnings: [],
  };

  // Check manifest
  const manifestLink = document.querySelector('link[rel="manifest"]');
  if (manifestLink) {
    results.passed.push('Manifest link found');

    try {
      const response = await fetch(manifestLink.href);
      const manifest = await response.json();

      // Required fields
      if (manifest.name) results.passed.push('Manifest has name');
      else results.failed.push('Manifest missing name');

      if (manifest.short_name) results.passed.push('Manifest has short_name');
      else results.failed.push('Manifest missing short_name');

      if (manifest.start_url) results.passed.push('Manifest has start_url');
      else results.failed.push('Manifest missing start_url');

      if (manifest.display === 'standalone' || manifest.display === 'fullscreen') {
        results.passed.push('Manifest has correct display mode');
      } else {
        results.failed.push('Manifest display should be standalone or fullscreen');
      }

      // Icons
      const icons = manifest.icons || [];
      const has192 = icons.some(i => i.sizes?.includes('192'));
      const has512 = icons.some(i => i.sizes?.includes('512'));
      const hasMaskable = icons.some(i => i.purpose?.includes('maskable'));

      if (has192) results.passed.push('Has 192x192 icon');
      else results.failed.push('Missing 192x192 icon');

      if (has512) results.passed.push('Has 512x512 icon');
      else results.failed.push('Missing 512x512 icon');

      if (hasMaskable) results.passed.push('Has maskable icon');
      else results.warnings.push('Missing maskable icon (recommended)');

    } catch (e) {
      results.failed.push('Failed to fetch/parse manifest: ' + e.message);
    }
  } else {
    results.failed.push('No manifest link found');
  }

  // Check service worker
  if ('serviceWorker' in navigator) {
    const registrations = await navigator.serviceWorker.getRegistrations();
    if (registrations.length > 0) {
      results.passed.push('Service worker registered');

      const active = registrations[0].active;
      if (active) {
        results.passed.push('Service worker is active');
      } else {
        results.warnings.push('Service worker not yet active');
      }
    } else {
      results.failed.push('No service worker registered');
    }
  } else {
    results.failed.push('Service workers not supported');
  }

  // Check HTTPS
  if (location.protocol === 'https:' || location.hostname === 'localhost') {
    results.passed.push('Running on HTTPS (or localhost)');
  } else {
    results.failed.push('Not running on HTTPS');
  }

  // Check viewport
  const viewport = document.querySelector('meta[name="viewport"]');
  if (viewport) {
    results.passed.push('Viewport meta tag present');
    if (viewport.content.includes('viewport-fit=cover')) {
      results.passed.push('viewport-fit=cover is set');
    } else {
      results.warnings.push('viewport-fit=cover not set (needed for iOS safe areas)');
    }
  } else {
    results.failed.push('Missing viewport meta tag');
  }

  // Check iOS meta tags
  const appleCapable = document.querySelector('meta[name="apple-mobile-web-app-capable"]');
  if (appleCapable) {
    results.passed.push('apple-mobile-web-app-capable meta tag present');
  } else {
    results.warnings.push('Missing apple-mobile-web-app-capable meta tag');
  }

  const appleTouchIcon = document.querySelector('link[rel="apple-touch-icon"]');
  if (appleTouchIcon) {
    results.passed.push('apple-touch-icon present');
  } else {
    results.warnings.push('Missing apple-touch-icon');
  }

  // Check theme color
  const themeColor = document.querySelector('meta[name="theme-color"]');
  if (themeColor) {
    results.passed.push('Theme color meta tag present');
  } else {
    results.warnings.push('Missing theme-color meta tag');
  }

  // Output results
  console.log('\n=== PWA Validation Results ===\n');

  console.log('✅ PASSED (' + results.passed.length + ')');
  results.passed.forEach(p => console.log('   ' + p));

  console.log('\n❌ FAILED (' + results.failed.length + ')');
  results.failed.forEach(f => console.log('   ' + f));

  console.log('\n⚠️  WARNINGS (' + results.warnings.length + ')');
  results.warnings.forEach(w => console.log('   ' + w));

  console.log('\n');

  if (results.failed.length === 0) {
    console.log('🎉 PWA meets minimum requirements!');
  } else {
    console.log('🔧 Fix the failed items above to meet PWA requirements.');
  }

  return results;
}

validatePWA();
```

---

## 9. Testing Checklist

### Development Testing
- [ ] Test with Chrome DevTools "Offline" mode
- [ ] Test with throttled network (Slow 3G)
- [ ] Test service worker updates
- [ ] Test cache clearing and reload

### Device Testing
- [ ] Test on Android Chrome
- [ ] Test on iOS Safari
- [ ] Test on desktop Chrome/Edge
- [ ] Test on tablet (if applicable)

### Installation Testing
- [ ] Install from Chrome address bar icon
- [ ] Install from browser menu
- [ ] Install from custom A2HS prompt
- [ ] Verify home screen icon appearance
- [ ] Verify splash screen
- [ ] Launch installed app and verify standalone mode

### Offline Testing
- [ ] Enable airplane mode and use app
- [ ] Submit form while offline, verify queuing
- [ ] Come back online, verify sync
- [ ] Test with intermittent connection

---

## 10. Final Sign-off

Before deployment, confirm:

- [ ] All critical checklist items pass
- [ ] Lighthouse PWA score is 90+
- [ ] Tested on at least one Android and one iOS device
- [ ] Offline mode works for core functionality
- [ ] A2HS prompt displays correctly
- [ ] Update flow works correctly
- [ ] No console errors in production build

**PWA is ready for deployment: ____________________**

**Date: ____________________**

**Validated by: ____________________**
