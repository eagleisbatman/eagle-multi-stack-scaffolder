# Next.js + Konsta UI Reference (PWA)

## Research Queries
- "Next.js PWA best practices 2025 2026"
- "Next.js 15 service worker Workbox setup"
- "Konsta UI Next.js mobile components"
- "Next.js offline storage IndexedDB"
- "Next.js PWA iOS Safari App Router"

## Package Manager
**Bun** - Fast installs, excellent Next.js support.

```bash
bunx create-next-app@latest my-pwa --typescript --tailwind --eslint --app --src-dir
```

## Why Next.js + Konsta UI for PWA

Choose Next.js + Konsta UI when:

1. **SEO Critical**: Marketing pages, e-commerce, content sites
2. **React Team**: Already working with React/Next.js
3. **SSR Required**: Server-side rendering for performance/SEO
4. **Simpler Offline**: Read-heavy apps, content caching
5. **Tailwind Stack**: Using Tailwind CSS already
6. **Existing Next.js**: Adding PWA features to existing app

Konsta UI provides iOS and Material Design components built on Tailwind CSS.

## Project Structure

```
my-pwa/
├── public/
│   ├── manifest.json
│   ├── sw.js                     # Service worker (generated)
│   └── icons/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── loading.tsx
│   │   ├── offline/
│   │   │   └── page.tsx
│   │   └── api/
│   │       └── [...route]/route.ts
│   ├── components/
│   │   ├── ui/
│   │   │   ├── SkeletonCard.tsx
│   │   │   └── EmptyState.tsx
│   │   └── pwa/
│   │       ├── AddToHomeScreen.tsx
│   │       ├── UpdatePrompt.tsx
│   │       ├── OfflineIndicator.tsx
│   │       └── PWAProvider.tsx
│   ├── hooks/
│   │   ├── useAddToHomeScreen.ts
│   │   ├── useNetworkStatus.ts
│   │   ├── useOfflineSync.ts
│   │   └── usePullToRefresh.ts
│   ├── lib/
│   │   ├── offline/
│   │   │   ├── db.ts
│   │   │   └── sync-manager.ts
│   │   └── utils.ts
│   └── styles/
│       └── globals.css
├── next.config.ts
├── tailwind.config.ts
├── package.json
└── tsconfig.json
```

## Essential Libraries

```bash
# PWA
bun add next-pwa
bun add -D workbox-webpack-plugin

# Konsta UI
bun add konsta

# Offline storage
bun add dexie

# State management
bun add zustand @tanstack/react-query

# Utilities
bun add clsx tailwind-merge date-fns uuid
bun add -D @types/uuid
```

## Next.js Config with PWA

```typescript
// next.config.ts
import type { NextConfig } from 'next';
import withPWA from 'next-pwa';

const nextConfig: NextConfig = {
  reactStrictMode: true,
};

const pwaConfig = withPWA({
  dest: 'public',
  register: true,
  skipWaiting: true,
  disable: process.env.NODE_ENV === 'development',
  runtimeCaching: [
    {
      urlPattern: /^https:\/\/fonts\.googleapis\.com\/.*/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'google-fonts-cache',
        expiration: {
          maxEntries: 10,
          maxAgeSeconds: 60 * 60 * 24 * 365,
        },
      },
    },
    {
      urlPattern: /^https:\/\/fonts\.gstatic\.com\/.*/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'gstatic-fonts-cache',
        expiration: {
          maxEntries: 10,
          maxAgeSeconds: 60 * 60 * 24 * 365,
        },
      },
    },
    {
      urlPattern: /\/api\/.*/i,
      handler: 'NetworkFirst',
      options: {
        cacheName: 'api-cache',
        networkTimeoutSeconds: 10,
        expiration: {
          maxEntries: 50,
          maxAgeSeconds: 60 * 60 * 24,
        },
      },
    },
    {
      urlPattern: /\.(?:png|jpg|jpeg|svg|gif|webp|ico)$/i,
      handler: 'CacheFirst',
      options: {
        cacheName: 'images-cache',
        expiration: {
          maxEntries: 100,
          maxAgeSeconds: 60 * 60 * 24 * 30,
        },
      },
    },
  ],
});

export default pwaConfig(nextConfig);
```

## Web App Manifest

```json
// public/manifest.json
{
  "name": "My Next.js PWA",
  "short_name": "MyPWA",
  "description": "A Progressive Web App built with Next.js and Konsta UI",
  "start_url": "/",
  "display": "standalone",
  "orientation": "portrait-primary",
  "background_color": "#ffffff",
  "theme_color": "#007aff",
  "icons": [
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
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
      "src": "/screenshots/mobile.png",
      "sizes": "1080x1920",
      "type": "image/png",
      "form_factor": "narrow"
    },
    {
      "src": "/screenshots/desktop.png",
      "sizes": "1920x1080",
      "type": "image/png",
      "form_factor": "wide"
    }
  ]
}
```

## Root Layout with Konsta UI

```tsx
// src/app/layout.tsx
import type { Metadata, Viewport } from 'next';
import { Inter } from 'next/font/google';
import { App } from 'konsta/react';
import { PWAProvider } from '@/components/pwa/PWAProvider';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'My PWA',
  description: 'A Progressive Web App',
  manifest: '/manifest.json',
  appleWebApp: {
    capable: true,
    statusBarStyle: 'default',
    title: 'My PWA',
  },
};

export const viewport: Viewport = {
  themeColor: '#007aff',
  width: 'device-width',
  initialScale: 1,
  maximumScale: 1,
  userScalable: false,
  viewportFit: 'cover',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <head>
        <link rel="apple-touch-icon" href="/icons/apple-touch-icon.png" />
        <meta name="apple-mobile-web-app-capable" content="yes" />
        <meta name="apple-mobile-web-app-status-bar-style" content="default" />
      </head>
      <body className={inter.className}>
        <App theme="ios" safeAreas>
          <PWAProvider>
            {children}
          </PWAProvider>
        </App>
      </body>
    </html>
  );
}
```

## Tailwind Config for Konsta UI

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';
import konstaConfig from 'konsta/config';

const config: Config = konstaConfig({
  content: [
    './src/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  darkMode: 'class',
  theme: {
    extend: {},
  },
  plugins: [],
});

export default config;
```

## Global Styles with Safe Areas

```css
/* src/styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --safe-area-inset-top: env(safe-area-inset-top);
  --safe-area-inset-bottom: env(safe-area-inset-bottom);
  --safe-area-inset-left: env(safe-area-inset-left);
  --safe-area-inset-right: env(safe-area-inset-right);
}

/* Prevent overscroll bounce */
html {
  overscroll-behavior: none;
}

/* Smooth scrolling */
.scrollable {
  -webkit-overflow-scrolling: touch;
  overscroll-behavior: contain;
}

/* Touch targets */
button,
a,
[role="button"] {
  min-height: 44px;
  min-width: 44px;
}

/* Disable text selection on interactive elements */
button,
[role="button"] {
  -webkit-user-select: none;
  user-select: none;
}

/* PWA standalone mode adjustments */
@media (display-mode: standalone) {
  body {
    /* Hide any browser-specific UI */
  }
}
```

## PWA Provider Component

```tsx
// src/components/pwa/PWAProvider.tsx
'use client';

import { useEffect, useState, createContext, useContext, ReactNode } from 'react';
import { AddToHomeScreen } from './AddToHomeScreen';
import { UpdatePrompt } from './UpdatePrompt';
import { OfflineIndicator } from './OfflineIndicator';

interface PWAContextType {
  isOnline: boolean;
  isStandalone: boolean;
  canInstall: boolean;
}

const PWAContext = createContext<PWAContextType>({
  isOnline: true,
  isStandalone: false,
  canInstall: false,
});

export const usePWA = () => useContext(PWAContext);

export function PWAProvider({ children }: { children: ReactNode }) {
  const [isOnline, setIsOnline] = useState(true);
  const [isStandalone, setIsStandalone] = useState(false);
  const [canInstall, setCanInstall] = useState(false);

  useEffect(() => {
    setIsOnline(navigator.onLine);
    setIsStandalone(window.matchMedia('(display-mode: standalone)').matches);

    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    const handleInstallPrompt = () => setCanInstall(true);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    window.addEventListener('beforeinstallprompt', handleInstallPrompt);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
      window.removeEventListener('beforeinstallprompt', handleInstallPrompt);
    };
  }, []);

  return (
    <PWAContext.Provider value={{ isOnline, isStandalone, canInstall }}>
      <OfflineIndicator />
      {children}
      <AddToHomeScreen />
      <UpdatePrompt />
    </PWAContext.Provider>
  );
}
```

## Add to Home Screen Component

```tsx
// src/components/pwa/AddToHomeScreen.tsx
'use client';

import { useState, useEffect, useCallback } from 'react';
import { Sheet, Block, Button } from 'konsta/react';

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

export function AddToHomeScreen() {
  const [showPrompt, setShowPrompt] = useState(false);
  const [deferredPrompt, setDeferredPrompt] = useState<BeforeInstallPromptEvent | null>(null);

  useEffect(() => {
    if (typeof window === 'undefined') return;

    // Check if already installed
    if (window.matchMedia('(display-mode: standalone)').matches) {
      return;
    }

    // Check if dismissed recently
    const dismissed = localStorage.getItem('a2hs-dismissed');
    if (dismissed) {
      const daysSince = (Date.now() - parseInt(dismissed)) / (1000 * 60 * 60 * 24);
      if (daysSince < 7) return;
    }

    const handler = (e: Event) => {
      e.preventDefault();
      setDeferredPrompt(e as BeforeInstallPromptEvent);
      setShowPrompt(true);
    };

    window.addEventListener('beforeinstallprompt', handler);
    return () => window.removeEventListener('beforeinstallprompt', handler);
  }, []);

  const install = useCallback(async () => {
    if (!deferredPrompt) return;

    deferredPrompt.prompt();
    const { outcome } = await deferredPrompt.userChoice;

    if (outcome === 'accepted') {
      setShowPrompt(false);
    }
    setDeferredPrompt(null);
  }, [deferredPrompt]);

  const dismiss = useCallback(() => {
    setShowPrompt(false);
    localStorage.setItem('a2hs-dismissed', Date.now().toString());
  }, []);

  return (
    <Sheet
      opened={showPrompt}
      onBackdropClick={dismiss}
    >
      <Block className="text-center">
        <p className="text-lg font-semibold mb-2">Install App</p>
        <p className="text-gray-600 dark:text-gray-400 mb-4">
          Install this app for a better experience with offline access.
        </p>
        <div className="flex gap-2 justify-center">
          <Button onClick={dismiss} outline>
            Later
          </Button>
          <Button onClick={install}>
            Install
          </Button>
        </div>
      </Block>
    </Sheet>
  );
}
```

## Update Prompt Component

```tsx
// src/components/pwa/UpdatePrompt.tsx
'use client';

import { useState, useEffect } from 'react';
import { Dialog, DialogButton } from 'konsta/react';

export function UpdatePrompt() {
  const [showUpdate, setShowUpdate] = useState(false);

  useEffect(() => {
    if (typeof window === 'undefined' || !('serviceWorker' in navigator)) return;

    navigator.serviceWorker.ready.then((registration) => {
      registration.addEventListener('updatefound', () => {
        const newWorker = registration.installing;
        if (!newWorker) return;

        newWorker.addEventListener('statechange', () => {
          if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
            setShowUpdate(true);
          }
        });
      });
    });

    // Reload on controller change
    let refreshing = false;
    navigator.serviceWorker.addEventListener('controllerchange', () => {
      if (!refreshing) {
        refreshing = true;
        window.location.reload();
      }
    });
  }, []);

  const handleUpdate = () => {
    navigator.serviceWorker.ready.then((registration) => {
      if (registration.waiting) {
        registration.waiting.postMessage({ type: 'SKIP_WAITING' });
      }
    });
  };

  return (
    <Dialog
      opened={showUpdate}
      title="Update Available"
      content="A new version is available. Update now for the latest features and improvements."
      buttons={
        <>
          <DialogButton onClick={() => setShowUpdate(false)}>
            Later
          </DialogButton>
          <DialogButton strong onClick={handleUpdate}>
            Update
          </DialogButton>
        </>
      }
    />
  );
}
```

## Offline Indicator Component

```tsx
// src/components/pwa/OfflineIndicator.tsx
'use client';

import { useState, useEffect } from 'react';
import { Notification, Icon } from 'konsta/react';
import { usePWA } from './PWAProvider';
import { db } from '@/lib/offline/db';

export function OfflineIndicator() {
  const { isOnline } = usePWA();
  const [pendingCount, setPendingCount] = useState(0);

  useEffect(() => {
    const updateCount = async () => {
      const count = await db.syncQueue.count();
      setPendingCount(count);
    };

    updateCount();
    const interval = setInterval(updateCount, 5000);
    return () => clearInterval(interval);
  }, []);

  if (isOnline) return null;

  return (
    <Notification
      opened={!isOnline}
      title="You're offline"
      subtitle={pendingCount > 0 ? `${pendingCount} changes pending sync` : undefined}
      className="fixed top-safe-top left-4 right-4 z-50"
    />
  );
}
```

## Pull to Refresh Hook

```tsx
// src/hooks/usePullToRefresh.ts
'use client';

import { useRef, useEffect, useState, useCallback } from 'react';

interface UsePullToRefreshOptions {
  onRefresh: () => Promise<void>;
  threshold?: number;
}

export function usePullToRefresh({ onRefresh, threshold = 80 }: UsePullToRefreshOptions) {
  const [isRefreshing, setIsRefreshing] = useState(false);
  const [pullDistance, setPullDistance] = useState(0);
  const containerRef = useRef<HTMLDivElement>(null);
  const startY = useRef(0);
  const isPulling = useRef(false);

  const handleTouchStart = useCallback((e: TouchEvent) => {
    if (containerRef.current?.scrollTop === 0) {
      startY.current = e.touches[0].clientY;
      isPulling.current = true;
    }
  }, []);

  const handleTouchMove = useCallback((e: TouchEvent) => {
    if (!isPulling.current) return;

    const currentY = e.touches[0].clientY;
    const distance = Math.max(0, currentY - startY.current);

    if (distance > 0 && containerRef.current?.scrollTop === 0) {
      e.preventDefault();
      setPullDistance(Math.min(distance, threshold * 1.5));
    }
  }, [threshold]);

  const handleTouchEnd = useCallback(async () => {
    if (!isPulling.current) return;

    isPulling.current = false;

    if (pullDistance >= threshold) {
      setIsRefreshing(true);
      try {
        await onRefresh();
      } finally {
        setIsRefreshing(false);
      }
    }

    setPullDistance(0);
  }, [pullDistance, threshold, onRefresh]);

  useEffect(() => {
    const container = containerRef.current;
    if (!container) return;

    container.addEventListener('touchstart', handleTouchStart, { passive: true });
    container.addEventListener('touchmove', handleTouchMove, { passive: false });
    container.addEventListener('touchend', handleTouchEnd);

    return () => {
      container.removeEventListener('touchstart', handleTouchStart);
      container.removeEventListener('touchmove', handleTouchMove);
      container.removeEventListener('touchend', handleTouchEnd);
    };
  }, [handleTouchStart, handleTouchMove, handleTouchEnd]);

  return {
    containerRef,
    isRefreshing,
    pullDistance,
    pullProgress: Math.min(pullDistance / threshold, 1),
  };
}
```

## Page with Pull to Refresh

```tsx
// src/app/page.tsx
'use client';

import { useState, useEffect } from 'react';
import {
  Page,
  Navbar,
  List,
  ListItem,
  Preloader,
  Block,
} from 'konsta/react';
import { usePullToRefresh } from '@/hooks/usePullToRefresh';

interface Item {
  id: string;
  name: string;
}

export default function HomePage() {
  const [loading, setLoading] = useState(true);
  const [items, setItems] = useState<Item[]>([]);

  const loadData = async () => {
    setLoading(true);
    try {
      // Load from API or local DB
      await new Promise((r) => setTimeout(r, 1000)); // Simulated
      setItems([
        { id: '1', name: 'Item 1' },
        { id: '2', name: 'Item 2' },
        { id: '3', name: 'Item 3' },
      ]);
    } finally {
      setLoading(false);
    }
  };

  const { containerRef, isRefreshing, pullProgress } = usePullToRefresh({
    onRefresh: loadData,
  });

  useEffect(() => {
    loadData();
  }, []);

  return (
    <Page>
      <Navbar title="Home" />

      {/* Pull indicator */}
      {pullProgress > 0 && (
        <div
          className="flex justify-center items-center transition-all"
          style={{ height: `${pullProgress * 60}px` }}
        >
          <Preloader />
        </div>
      )}

      <div ref={containerRef} className="overflow-auto h-full">
        {isRefreshing && (
          <Block className="text-center">
            <Preloader />
            <p className="mt-2">Refreshing...</p>
          </Block>
        )}

        {loading && !isRefreshing ? (
          <List>
            {[1, 2, 3, 4, 5].map((i) => (
              <ListItem key={i} title={<div className="skeleton-text w-32 h-4" />} />
            ))}
          </List>
        ) : (
          <List>
            {items.map((item) => (
              <ListItem key={item.id} title={item.name} link />
            ))}
          </List>
        )}
      </div>
    </Page>
  );
}
```

## Dexie Database Setup

```typescript
// src/lib/offline/db.ts
import Dexie, { Table } from 'dexie';

export interface SyncableRecord {
  id: string;
  _synced: boolean;
  _syncedAt?: number;
  _localUpdatedAt: number;
  _deleted?: boolean;
}

export interface SyncQueueItem {
  id?: number;
  operation: 'create' | 'update' | 'delete';
  table: string;
  recordId: string;
  data: Record<string, unknown>;
  createdAt: number;
  retryCount: number;
}

export class AppDatabase extends Dexie {
  syncQueue!: Table<SyncQueueItem, number>;

  constructor() {
    super('NextPWADatabase');

    this.version(1).stores({
      syncQueue: '++id, table, recordId, createdAt',
    });
  }
}

export const db = new AppDatabase();
```

## Offline Page

```tsx
// src/app/offline/page.tsx
import { Page, Navbar, Block, Icon } from 'konsta/react';

export default function OfflinePage() {
  return (
    <Page>
      <Navbar title="Offline" />
      <Block className="text-center mt-20">
        <div className="text-6xl mb-4">📴</div>
        <h2 className="text-xl font-semibold mb-2">You're Offline</h2>
        <p className="text-gray-600 dark:text-gray-400">
          Please check your internet connection and try again.
        </p>
      </Block>
    </Page>
  );
}
```

## Setup Commands

```bash
# Create Next.js project
bunx create-next-app@latest my-pwa --typescript --tailwind --eslint --app --src-dir

cd my-pwa

# Install PWA and Konsta UI
bun add next-pwa konsta
bun add -D workbox-webpack-plugin

# Install offline and state management
bun add dexie zustand @tanstack/react-query

# Install utilities
bun add clsx tailwind-merge date-fns uuid
bun add -D @types/uuid

# Update next.config.ts with PWA config
# Update tailwind.config.ts with Konsta

# Create manifest.json in public/
# Generate PWA icons

# Development
bun dev

# Build
bun run build

# Start production
bun start
```

## Key Rules

1. **Use App Router metadata** for PWA meta tags
2. **Wrap with Konsta App** for proper styling
3. **Enable safeAreas** on Konsta App component
4. **Test on real iOS devices** - Safari has many PWA quirks
5. **Use server components** for SEO pages
6. **Use client components** for interactive PWA features
7. **Handle SSR gracefully** - Check `typeof window` before using browser APIs
8. **Cache SSR pages** - Use Next.js caching for offline support
