# Add to Home Screen (A2HS) Prompt Component

## Overview

The "Add to Home Screen" prompt encourages users to install your PWA. Browsers show a native prompt, but you can enhance the experience with a custom UI that triggers it.

## Browser Support

| Browser | Auto Prompt | `beforeinstallprompt` Event |
|---------|-------------|---------------------------|
| Chrome (Android) | Yes | Yes |
| Chrome (Desktop) | Yes | Yes |
| Edge | Yes | Yes |
| Safari (iOS) | No | No |
| Firefox | No | No |

**Important**: iOS Safari does NOT support `beforeinstallprompt`. You must show manual instructions.

## Requirements for Install Prompt

The browser will fire `beforeinstallprompt` when:

1. HTTPS connection (or localhost)
2. Valid web manifest with:
   - `name` or `short_name`
   - `icons` (192px and 512px PNG)
   - `start_url`
   - `display: standalone` or `fullscreen`
3. Service worker registered
4. User engagement heuristic met (varies by browser)
5. App not already installed

## React Implementation

```tsx
// components/pwa/AddToHomeScreen.tsx
'use client';

import { useState, useEffect, useCallback } from 'react';

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

interface AddToHomeScreenProps {
  /** Delay before showing prompt (ms) */
  delay?: number;
  /** Days to wait before showing again after dismiss */
  dismissDays?: number;
  /** Custom render prop for the prompt UI */
  children?: (props: {
    show: boolean;
    install: () => void;
    dismiss: () => void;
    isIOS: boolean;
  }) => React.ReactNode;
}

export function AddToHomeScreen({
  delay = 3000,
  dismissDays = 7,
  children,
}: AddToHomeScreenProps) {
  const [showPrompt, setShowPrompt] = useState(false);
  const [showIOSInstructions, setShowIOSInstructions] = useState(false);
  const [deferredPrompt, setDeferredPrompt] =
    useState<BeforeInstallPromptEvent | null>(null);

  const isIOS =
    typeof navigator !== 'undefined' &&
    /iPad|iPhone|iPod/.test(navigator.userAgent);

  const isStandalone =
    typeof window !== 'undefined' &&
    (window.matchMedia('(display-mode: standalone)').matches ||
      (navigator as any).standalone);

  useEffect(() => {
    // Don't show if already installed
    if (isStandalone) return;

    // Check if dismissed recently
    const dismissed = localStorage.getItem('a2hs-dismissed');
    if (dismissed) {
      const dismissedAt = parseInt(dismissed, 10);
      const daysSince = (Date.now() - dismissedAt) / (1000 * 60 * 60 * 24);
      if (daysSince < dismissDays) return;
    }

    // For iOS, show instructions after delay
    if (isIOS) {
      const timer = setTimeout(() => {
        setShowIOSInstructions(true);
      }, delay);
      return () => clearTimeout(timer);
    }

    // For Chrome/Edge, listen for beforeinstallprompt
    const handler = (e: Event) => {
      e.preventDefault();
      setDeferredPrompt(e as BeforeInstallPromptEvent);

      // Show prompt after delay
      setTimeout(() => {
        setShowPrompt(true);
      }, delay);
    };

    window.addEventListener('beforeinstallprompt', handler);

    // Check if prompt was already captured before component mounted
    if ((window as any).deferredPrompt) {
      setDeferredPrompt((window as any).deferredPrompt);
      setTimeout(() => setShowPrompt(true), delay);
    }

    return () => {
      window.removeEventListener('beforeinstallprompt', handler);
    };
  }, [delay, dismissDays, isIOS, isStandalone]);

  const install = useCallback(async () => {
    if (!deferredPrompt) return;

    // Show browser's install prompt
    deferredPrompt.prompt();

    // Wait for user choice
    const { outcome } = await deferredPrompt.userChoice;

    if (outcome === 'accepted') {
      console.log('PWA installed');
      // Track installation
      if (typeof gtag !== 'undefined') {
        gtag('event', 'pwa_install', { method: 'a2hs_prompt' });
      }
    }

    setDeferredPrompt(null);
    setShowPrompt(false);
  }, [deferredPrompt]);

  const dismiss = useCallback(() => {
    setShowPrompt(false);
    setShowIOSInstructions(false);
    localStorage.setItem('a2hs-dismissed', Date.now().toString());
  }, []);

  // Custom render prop
  if (children) {
    return (
      <>
        {children({
          show: showPrompt || showIOSInstructions,
          install,
          dismiss,
          isIOS,
        })}
      </>
    );
  }

  // Default UI
  if (showIOSInstructions) {
    return <IOSInstructions onDismiss={dismiss} />;
  }

  if (!showPrompt) return null;

  return (
    <div className="a2hs-banner">
      <div className="a2hs-content">
        <img src="/icons/icon-64.png" alt="App icon" className="a2hs-icon" />
        <div className="a2hs-text">
          <strong>Install App</strong>
          <span>Add to home screen for the best experience</span>
        </div>
      </div>
      <div className="a2hs-actions">
        <button onClick={dismiss} className="a2hs-dismiss">
          Not now
        </button>
        <button onClick={install} className="a2hs-install">
          Install
        </button>
      </div>
    </div>
  );
}

// iOS-specific instructions
function IOSInstructions({ onDismiss }: { onDismiss: () => void }) {
  return (
    <div className="ios-instructions">
      <button onClick={onDismiss} className="ios-close">
        ×
      </button>
      <h3>Install this app</h3>
      <p>
        Tap{' '}
        <span className="ios-share-icon">
          <ShareIcon />
        </span>{' '}
        then "Add to Home Screen"
      </p>
      <div className="ios-arrow">↓</div>
    </div>
  );
}

function ShareIcon() {
  return (
    <svg
      width="20"
      height="20"
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth="2"
    >
      <path d="M4 12v8a2 2 0 002 2h12a2 2 0 002-2v-8" />
      <polyline points="16 6 12 2 8 6" />
      <line x1="12" y1="2" x2="12" y2="15" />
    </svg>
  );
}
```

## CSS Styles

```css
/* Add to Home Screen Banner */
.a2hs-banner {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 9999;

  background: white;
  border-top: 1px solid #e5e7eb;
  padding: 16px;
  padding-bottom: calc(16px + env(safe-area-inset-bottom));

  display: flex;
  flex-direction: column;
  gap: 12px;

  box-shadow: 0 -4px 12px rgba(0, 0, 0, 0.1);
  animation: slideUp 0.3s ease;
}

@keyframes slideUp {
  from {
    transform: translateY(100%);
  }
  to {
    transform: translateY(0);
  }
}

.a2hs-content {
  display: flex;
  align-items: center;
  gap: 12px;
}

.a2hs-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
}

.a2hs-text {
  display: flex;
  flex-direction: column;
}

.a2hs-text strong {
  font-size: 16px;
  font-weight: 600;
}

.a2hs-text span {
  font-size: 14px;
  color: #6b7280;
}

.a2hs-actions {
  display: flex;
  gap: 8px;
}

.a2hs-dismiss,
.a2hs-install {
  flex: 1;
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.15s;
}

.a2hs-dismiss {
  background: transparent;
  border: 1px solid #d1d5db;
  color: #374151;
}

.a2hs-dismiss:active {
  background: #f3f4f6;
}

.a2hs-install {
  background: #3b82f6;
  border: none;
  color: white;
}

.a2hs-install:active {
  background: #2563eb;
}

/* iOS Instructions */
.ios-instructions {
  position: fixed;
  bottom: 20px;
  left: 20px;
  right: 20px;
  z-index: 9999;

  background: white;
  border-radius: 16px;
  padding: 20px;
  text-align: center;

  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.ios-close {
  position: absolute;
  top: 8px;
  right: 12px;
  background: none;
  border: none;
  font-size: 24px;
  color: #9ca3af;
  cursor: pointer;
}

.ios-instructions h3 {
  margin: 0 0 8px;
  font-size: 18px;
}

.ios-instructions p {
  margin: 0;
  color: #6b7280;
}

.ios-share-icon {
  display: inline-flex;
  vertical-align: middle;
  color: #3b82f6;
}

.ios-arrow {
  font-size: 24px;
  margin-top: 12px;
  animation: bounce 1s infinite;
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(5px);
  }
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  .a2hs-banner {
    background: #1f2937;
    border-top-color: #374151;
  }

  .a2hs-text span {
    color: #9ca3af;
  }

  .a2hs-dismiss {
    border-color: #4b5563;
    color: #e5e7eb;
  }

  .a2hs-dismiss:active {
    background: #374151;
  }

  .ios-instructions {
    background: #1f2937;
    color: white;
  }

  .ios-instructions p {
    color: #9ca3af;
  }

  .ios-close {
    color: #6b7280;
  }
}
```

## Vue Implementation

```vue
<!-- components/pwa/AddToHomeScreen.vue -->
<template>
  <div v-if="showIOSInstructions" class="ios-instructions">
    <button @click="dismiss" class="ios-close">×</button>
    <h3>Install this app</h3>
    <p>
      Tap
      <span class="ios-share-icon">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M4 12v8a2 2 0 002 2h12a2 2 0 002-2v-8" />
          <polyline points="16 6 12 2 8 6" />
          <line x1="12" y1="2" x2="12" y2="15" />
        </svg>
      </span>
      then "Add to Home Screen"
    </p>
    <div class="ios-arrow">↓</div>
  </div>

  <div v-else-if="showPrompt" class="a2hs-banner">
    <div class="a2hs-content">
      <img src="/icons/icon-64.png" alt="App icon" class="a2hs-icon" />
      <div class="a2hs-text">
        <strong>Install App</strong>
        <span>Add to home screen for the best experience</span>
      </div>
    </div>
    <div class="a2hs-actions">
      <button @click="dismiss" class="a2hs-dismiss">Not now</button>
      <button @click="install" class="a2hs-install">Install</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

const props = withDefaults(defineProps<{
  delay?: number;
  dismissDays?: number;
}>(), {
  delay: 3000,
  dismissDays: 7,
});

const showPrompt = ref(false);
const showIOSInstructions = ref(false);
const deferredPrompt = ref<BeforeInstallPromptEvent | null>(null);

const isIOS = computed(() =>
  typeof navigator !== 'undefined' && /iPad|iPhone|iPod/.test(navigator.userAgent)
);

const isStandalone = computed(() =>
  typeof window !== 'undefined' &&
  (window.matchMedia('(display-mode: standalone)').matches ||
    (navigator as any).standalone)
);

onMounted(() => {
  if (isStandalone.value) return;

  const dismissed = localStorage.getItem('a2hs-dismissed');
  if (dismissed) {
    const daysSince = (Date.now() - parseInt(dismissed)) / (1000 * 60 * 60 * 24);
    if (daysSince < props.dismissDays) return;
  }

  if (isIOS.value) {
    setTimeout(() => {
      showIOSInstructions.value = true;
    }, props.delay);
    return;
  }

  const handler = (e: Event) => {
    e.preventDefault();
    deferredPrompt.value = e as BeforeInstallPromptEvent;
    setTimeout(() => {
      showPrompt.value = true;
    }, props.delay);
  };

  window.addEventListener('beforeinstallprompt', handler);
});

async function install() {
  if (!deferredPrompt.value) return;

  deferredPrompt.value.prompt();
  const { outcome } = await deferredPrompt.value.userChoice;

  if (outcome === 'accepted') {
    console.log('PWA installed');
  }

  deferredPrompt.value = null;
  showPrompt.value = false;
}

function dismiss() {
  showPrompt.value = false;
  showIOSInstructions.value = false;
  localStorage.setItem('a2hs-dismissed', Date.now().toString());
}
</script>
```

## Global Event Capture

Capture the `beforeinstallprompt` event early in your app's lifecycle:

```typescript
// main.ts or early in app initialization
if (typeof window !== 'undefined') {
  window.addEventListener('beforeinstallprompt', (e) => {
    e.preventDefault();
    // Store for later use
    (window as any).deferredPrompt = e;
  });

  // Track when app is installed
  window.addEventListener('appinstalled', () => {
    console.log('PWA installed');
    (window as any).deferredPrompt = null;

    // Track installation
    if (typeof gtag !== 'undefined') {
      gtag('event', 'pwa_install', { method: 'browser_prompt' });
    }
  });
}
```

## Analytics Tracking

```typescript
// Track A2HS interactions
function trackA2HS(action: 'shown' | 'installed' | 'dismissed') {
  // Google Analytics 4
  if (typeof gtag !== 'undefined') {
    gtag('event', 'a2hs_prompt', {
      action,
      platform: isIOS() ? 'ios' : 'other',
    });
  }

  // Plausible
  if (typeof plausible !== 'undefined') {
    plausible('A2HS', { props: { action } });
  }
}
```

## Best Practices

1. **Don't show immediately** - Wait for user engagement
2. **Respect dismissal** - Don't annoy users, wait days before reshowing
3. **Handle iOS separately** - Manual instructions are required
4. **Track installations** - Measure conversion rate
5. **Test the flow** - Use Chrome DevTools Application panel
6. **Explain the benefit** - Tell users why they should install
7. **Make it dismissible** - Always provide a "not now" option
8. **Show in context** - Trigger after a valuable action (completed task, etc.)
