# iOS Safe Area Handling

## Overview

iOS devices with notches (iPhone X+) and home indicators require special handling to prevent content from being obscured. Safe areas are the regions of the screen that are guaranteed to be visible.

## Safe Area Insets

```
┌─────────────────────────────────┐
│         Status Bar              │  ← safe-area-inset-top
│  ┌───────────────────────────┐  │
│  │                           │  │
│  │                           │  │
│  │      Safe Content         │  │
│  │         Area              │  │
│  │                           │  │
│  │                           │  │
│  └───────────────────────────┘  │
│        ════════════════         │  ← safe-area-inset-bottom (home indicator)
└─────────────────────────────────┘

← safe-area-inset-left    safe-area-inset-right →
      (landscape)              (landscape)
```

## CSS Environment Variables

```css
/* The env() function provides safe area values */
padding-top: env(safe-area-inset-top);
padding-right: env(safe-area-inset-right);
padding-bottom: env(safe-area-inset-bottom);
padding-left: env(safe-area-inset-left);

/* With fallbacks for older browsers */
padding-top: env(safe-area-inset-top, 0px);
padding-bottom: env(safe-area-inset-bottom, 0px);
```

## Viewport Configuration

**Required** for safe areas to work:

```html
<meta
  name="viewport"
  content="width=device-width, initial-scale=1.0, viewport-fit=cover"
>
```

**Key**: `viewport-fit=cover` makes the app extend into safe areas, requiring manual handling.

## Complete CSS Template

```css
/* ===========================================
   ROOT CSS VARIABLES
   =========================================== */

:root {
  /* Safe area CSS custom properties for easier use */
  --safe-top: env(safe-area-inset-top, 0px);
  --safe-right: env(safe-area-inset-right, 0px);
  --safe-bottom: env(safe-area-inset-bottom, 0px);
  --safe-left: env(safe-area-inset-left, 0px);

  /* Common header/footer heights */
  --header-height: 56px;
  --footer-height: 56px;
  --tab-bar-height: 50px;
}

/* ===========================================
   HTML/BODY SETUP
   =========================================== */

html {
  /* Prevent overscroll bounce */
  overscroll-behavior: none;

  /* Full height */
  height: 100%;
}

body {
  /* Minimum height with safe area consideration */
  min-height: 100%;

  /* Extend background into safe areas */
  background-color: var(--background-color, #ffffff);

  /* Safe area padding (use this OR handle on child elements) */
  padding-top: var(--safe-top);
  padding-bottom: var(--safe-bottom);
  padding-left: var(--safe-left);
  padding-right: var(--safe-right);
}

/* Alternative: Only apply to root app container */
.app-root {
  min-height: 100vh;
  padding-top: var(--safe-top);
  padding-bottom: var(--safe-bottom);
}

/* ===========================================
   FIXED HEADER
   =========================================== */

.header-fixed {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;

  /* Include safe area in height */
  padding-top: var(--safe-top);
  height: calc(var(--header-height) + var(--safe-top));

  /* Left/right safe areas for landscape */
  padding-left: var(--safe-left);
  padding-right: var(--safe-right);

  background-color: var(--header-bg, #ffffff);
}

/* Content area when header is fixed */
.content-with-fixed-header {
  /* Push content below header */
  padding-top: calc(var(--header-height) + var(--safe-top));
}

/* ===========================================
   FIXED FOOTER / TAB BAR
   =========================================== */

.footer-fixed {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 1000;

  /* Include safe area in height */
  padding-bottom: var(--safe-bottom);
  height: calc(var(--footer-height) + var(--safe-bottom));

  /* Left/right safe areas for landscape */
  padding-left: var(--safe-left);
  padding-right: var(--safe-right);

  background-color: var(--footer-bg, #ffffff);
}

.tab-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 1000;

  /* Tab bar specific height */
  padding-bottom: var(--safe-bottom);
  height: calc(var(--tab-bar-height) + var(--safe-bottom));

  display: flex;
  justify-content: space-around;
  align-items: flex-start; /* Align items to top, padding handles bottom */

  background-color: var(--tab-bar-bg, #ffffff);
  border-top: 1px solid var(--border-color, #e5e5e5);
}

.tab-bar-item {
  /* Touch target */
  min-height: var(--tab-bar-height);
  min-width: 64px;
  padding: 8px 12px;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

/* Content area when footer/tab bar is fixed */
.content-with-fixed-footer {
  padding-bottom: calc(var(--footer-height) + var(--safe-bottom));
}

.content-with-tab-bar {
  padding-bottom: calc(var(--tab-bar-height) + var(--safe-bottom));
}

/* ===========================================
   FLOATING ACTION BUTTON (FAB)
   =========================================== */

.fab {
  position: fixed;
  z-index: 999;

  /* Default position: bottom right */
  bottom: calc(24px + var(--safe-bottom));
  right: calc(24px + var(--safe-right));

  /* When tab bar exists */
  &.fab-with-tab-bar {
    bottom: calc(var(--tab-bar-height) + var(--safe-bottom) + 16px);
  }
}

/* ===========================================
   MODAL / SHEET
   =========================================== */

.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 2000;
}

.modal-content {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 2001;

  /* Respect safe areas */
  max-height: calc(100vh - var(--safe-top) - var(--safe-bottom) - 48px);
  max-width: calc(100vw - var(--safe-left) - var(--safe-right) - 32px);
  overflow-y: auto;
}

/* Bottom sheet */
.bottom-sheet {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 2001;

  /* Include safe area padding */
  padding-bottom: var(--safe-bottom);
  padding-left: var(--safe-left);
  padding-right: var(--safe-right);

  /* Visual */
  background: white;
  border-radius: 16px 16px 0 0;
  max-height: calc(100vh - var(--safe-top) - 44px);
  overflow-y: auto;
}

/* ===========================================
   FULL-SCREEN PAGES
   =========================================== */

.page-fullscreen {
  position: fixed;
  inset: 0;
  z-index: 100;

  /* Safe area padding */
  padding-top: var(--safe-top);
  padding-bottom: var(--safe-bottom);
  padding-left: var(--safe-left);
  padding-right: var(--safe-right);

  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}

/* ===========================================
   KEYBOARD HANDLING
   =========================================== */

/* When keyboard is open, reduce bottom safe area */
.keyboard-open .footer-fixed,
.keyboard-open .tab-bar,
.keyboard-open .fab {
  /* Keyboard usually covers home indicator */
  padding-bottom: 0;
  bottom: 0;
}

.keyboard-open .bottom-sheet {
  /* Don't add extra padding when keyboard is open */
  padding-bottom: 8px;
}

/* ===========================================
   BANNERS / TOASTS
   =========================================== */

/* Top banner (offline indicator, etc.) */
.banner-top {
  position: fixed;
  top: var(--safe-top);
  left: var(--safe-left);
  right: var(--safe-right);
  z-index: 1100;
}

/* When header exists, position below it */
.banner-below-header {
  top: calc(var(--header-height) + var(--safe-top));
}

/* Bottom toast */
.toast-bottom {
  position: fixed;
  bottom: calc(16px + var(--safe-bottom));
  left: calc(16px + var(--safe-left));
  right: calc(16px + var(--safe-right));
  z-index: 3000;
}

/* Toast above tab bar */
.toast-above-tab-bar {
  bottom: calc(var(--tab-bar-height) + var(--safe-bottom) + 16px);
}

/* ===========================================
   LANDSCAPE ORIENTATION
   =========================================== */

@media (orientation: landscape) {
  .header-fixed,
  .footer-fixed {
    /* Ensure left/right safe areas are respected */
    padding-left: var(--safe-left);
    padding-right: var(--safe-right);
  }

  /* On phones, landscape has notch on left or right */
  .content-landscape {
    padding-left: var(--safe-left);
    padding-right: var(--safe-right);
  }
}

/* ===========================================
   iPAD SPECIFIC
   =========================================== */

@media (min-width: 768px) {
  /* iPad has minimal safe areas (only status bar) */
  /* Adjust spacing for larger screens */

  .modal-content {
    max-width: 600px;
    max-height: 80vh;
  }

  .bottom-sheet {
    max-width: 600px;
    left: 50%;
    transform: translateX(-50%);
    border-radius: 16px 16px 0 0;
  }
}

/* ===========================================
   PWA STANDALONE MODE
   =========================================== */

@media (display-mode: standalone) {
  /* App is running as installed PWA */

  /* Ensure full safe area handling */
  body {
    /* Additional PWA-specific styles */
  }

  /* Hide browser UI hints */
  .install-prompt {
    display: none;
  }
}
```

## JavaScript Helpers

```typescript
// safe-area-utils.ts

/**
 * Get current safe area inset values
 */
export function getSafeAreaInsets(): {
  top: number;
  right: number;
  bottom: number;
  left: number;
} {
  const computedStyle = getComputedStyle(document.documentElement);

  return {
    top: parseInt(computedStyle.getPropertyValue('--safe-top') || '0', 10),
    right: parseInt(computedStyle.getPropertyValue('--safe-right') || '0', 10),
    bottom: parseInt(computedStyle.getPropertyValue('--safe-bottom') || '0', 10),
    left: parseInt(computedStyle.getPropertyValue('--safe-left') || '0', 10),
  };
}

/**
 * Check if device has a notch
 */
export function hasNotch(): boolean {
  const insets = getSafeAreaInsets();
  return insets.top > 20 || insets.bottom > 0;
}

/**
 * Check if running in standalone mode (installed PWA)
 */
export function isStandalone(): boolean {
  return (
    window.matchMedia('(display-mode: standalone)').matches ||
    (window.navigator as any).standalone === true
  );
}

/**
 * Listen for keyboard show/hide (iOS)
 */
export function onKeyboardChange(
  callback: (isOpen: boolean, height: number) => void
): () => void {
  let lastHeight = window.innerHeight;

  const handleResize = () => {
    const currentHeight = window.innerHeight;
    const heightDiff = lastHeight - currentHeight;

    // Keyboard is considered open if height decreased by > 150px
    const isOpen = heightDiff > 150;
    callback(isOpen, isOpen ? heightDiff : 0);

    lastHeight = currentHeight;
  };

  // Also use visualViewport API if available
  if (window.visualViewport) {
    window.visualViewport.addEventListener('resize', () => {
      const isOpen = window.visualViewport!.height < window.innerHeight * 0.75;
      const keyboardHeight = window.innerHeight - window.visualViewport!.height;
      callback(isOpen, keyboardHeight);
    });
  }

  window.addEventListener('resize', handleResize);

  return () => {
    window.removeEventListener('resize', handleResize);
  };
}

/**
 * Scroll input into view above keyboard
 */
export function scrollInputIntoView(input: HTMLElement): void {
  setTimeout(() => {
    input.scrollIntoView({
      behavior: 'smooth',
      block: 'center',
    });
  }, 300); // Wait for keyboard animation
}
```

## React Hook

```typescript
// useSafeArea.ts
import { useState, useEffect } from 'react';

interface SafeAreaInsets {
  top: number;
  right: number;
  bottom: number;
  left: number;
}

export function useSafeArea(): SafeAreaInsets {
  const [insets, setInsets] = useState<SafeAreaInsets>({
    top: 0,
    right: 0,
    bottom: 0,
    left: 0,
  });

  useEffect(() => {
    const updateInsets = () => {
      const style = getComputedStyle(document.documentElement);
      setInsets({
        top: parseInt(style.getPropertyValue('--safe-top') || '0', 10),
        right: parseInt(style.getPropertyValue('--safe-right') || '0', 10),
        bottom: parseInt(style.getPropertyValue('--safe-bottom') || '0', 10),
        left: parseInt(style.getPropertyValue('--safe-left') || '0', 10),
      });
    };

    updateInsets();
    window.addEventListener('resize', updateInsets);
    window.addEventListener('orientationchange', updateInsets);

    return () => {
      window.removeEventListener('resize', updateInsets);
      window.removeEventListener('orientationchange', updateInsets);
    };
  }, []);

  return insets;
}
```

## Vue Composable

```typescript
// useSafeArea.ts
import { ref, onMounted, onUnmounted } from 'vue';

export function useSafeArea() {
  const insets = ref({
    top: 0,
    right: 0,
    bottom: 0,
    left: 0,
  });

  const updateInsets = () => {
    const style = getComputedStyle(document.documentElement);
    insets.value = {
      top: parseInt(style.getPropertyValue('--safe-top') || '0', 10),
      right: parseInt(style.getPropertyValue('--safe-right') || '0', 10),
      bottom: parseInt(style.getPropertyValue('--safe-bottom') || '0', 10),
      left: parseInt(style.getPropertyValue('--safe-left') || '0', 10),
    };
  };

  onMounted(() => {
    updateInsets();
    window.addEventListener('resize', updateInsets);
    window.addEventListener('orientationchange', updateInsets);
  });

  onUnmounted(() => {
    window.removeEventListener('resize', updateInsets);
    window.removeEventListener('orientationchange', updateInsets);
  });

  return { insets };
}
```

## Testing Safe Areas

### Chrome DevTools
1. Open DevTools → Toggle device toolbar
2. Select iPhone X/12/13/14 preset
3. The emulator simulates safe areas

### Safari Simulator
1. Open Xcode → Simulator
2. Choose device with notch
3. Most accurate for iOS testing

### Real Device Testing
**Always test on real iOS devices** - emulators often miss edge cases.

## Common Issues

| Issue | Solution |
|-------|----------|
| Content behind notch | Add `viewport-fit=cover` meta tag |
| Safe area values are 0 | Check `viewport-fit=cover` is set |
| Footer clips on keyboard | Use `visualViewport` API to detect keyboard |
| Landscape notch overlap | Ensure left/right safe areas are respected |
| iPad looks wrong | Safe areas are minimal on iPad, use min-height |
