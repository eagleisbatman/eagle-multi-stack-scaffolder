# Ionic React Reference (PWA)

## Research Queries
- "Ionic React PWA best practices 2025 2026"
- "Ionic Capacitor React PWA hybrid"
- "Ionic React Vite service worker Workbox"
- "Ionic React offline storage IndexedDB"
- "React PWA iOS Safari limitations"

## Package Manager
**Bun** - Fast installs, fully compatible with Ionic CLI.

```bash
bunx @ionic/cli start my-pwa blank --type=react --capacitor
```

## Why Ionic React for PWA

Choose Ionic React when:

1. **React Team**: Your team already knows React
2. **Native Path**: You might need Capacitor for native features later
3. **iOS + Android Design**: Automatic platform-adaptive styling
4. **Rich Components**: Mobile-optimized UI components out of the box
5. **Existing React App**: Easier to integrate with existing React codebase

## Project Structure

```
my-pwa/
├── public/
│   ├── manifest.json
│   └── icons/
├── src/
│   ├── components/
│   │   ├── ui/
│   │   │   ├── SkeletonCard.tsx
│   │   │   └── EmptyState.tsx
│   │   └── pwa/
│   │       ├── AddToHomeScreen.tsx
│   │       ├── UpdatePrompt.tsx
│   │       └── OfflineIndicator.tsx
│   ├── hooks/
│   │   ├── useOfflineSync.ts
│   │   ├── useNetworkStatus.ts
│   │   └── usePullToRefresh.ts
│   ├── pages/
│   │   ├── Home.tsx
│   │   └── Offline.tsx
│   ├── lib/
│   │   └── offline/
│   │       ├── db.ts
│   │       └── sync-manager.ts
│   ├── stores/
│   │   └── sync-queue.ts
│   ├── theme/
│   │   └── variables.css
│   ├── App.tsx
│   └── main.tsx
├── capacitor.config.ts
├── vite.config.ts
├── package.json
└── tsconfig.json
```

## Essential Libraries

```bash
# PWA plugin for Vite
bun add -D vite-plugin-pwa workbox-window

# Offline storage
bun add dexie

# State management
bun add zustand @tanstack/react-query

# HTTP client
bun add axios axios-retry

# Utilities
bun add date-fns uuid
bun add -D @types/uuid
```

## Vite Config with PWA Plugin

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';
import path from 'path';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'prompt',
      includeAssets: ['favicon.ico', 'robots.txt', 'icons/*.png'],
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff,woff2}'],
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
              cacheableResponse: {
                statuses: [0, 200],
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
              cacheableResponse: {
                statuses: [0, 200],
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
              cacheableResponse: {
                statuses: [0, 200],
              },
            },
          },
        ],
      },
      manifest: {
        name: 'My Ionic React PWA',
        short_name: 'MyPWA',
        description: 'A Progressive Web App built with Ionic React',
        theme_color: '#3880ff',
        background_color: '#ffffff',
        display: 'standalone',
        orientation: 'portrait-primary',
        start_url: '/',
        icons: [
          {
            src: '/icons/icon-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: '/icons/icon-512x512.png',
            sizes: '512x512',
            type: 'image/png',
          },
          {
            src: '/icons/icon-512x512.png',
            sizes: '512x512',
            type: 'image/png',
            purpose: 'maskable',
          },
        ],
      },
    }),
  ],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

## PWA Registration

```typescript
// src/main.tsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import App from './App';

import '@ionic/react/css/core.css';
import '@ionic/react/css/normalize.css';
import '@ionic/react/css/structure.css';
import '@ionic/react/css/typography.css';
import '@ionic/react/css/padding.css';
import '@ionic/react/css/float-elements.css';
import '@ionic/react/css/text-alignment.css';
import '@ionic/react/css/text-transformation.css';
import '@ionic/react/css/flex-utils.css';
import '@ionic/react/css/display.css';
import './theme/variables.css';

import { registerSW } from 'virtual:pwa-register';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 60, // 1 hour
    },
  },
});

const container = document.getElementById('root');
const root = createRoot(container!);

root.render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </React.StrictMode>
);

// PWA Registration
const updateSW = registerSW({
  onNeedRefresh() {
    window.dispatchEvent(new CustomEvent('sw-update-available'));
  },
  onOfflineReady() {
    console.log('App ready to work offline');
  },
});

(window as any).updateSW = updateSW;
```

## Add to Home Screen Hook

```typescript
// src/hooks/useAddToHomeScreen.ts
import { useState, useEffect, useCallback } from 'react';

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

export function useAddToHomeScreen() {
  const [showPrompt, setShowPrompt] = useState(false);
  const [deferredPrompt, setDeferredPrompt] = useState<BeforeInstallPromptEvent | null>(null);

  useEffect(() => {
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

  return { showPrompt, install, dismiss };
}
```

## Add to Home Screen Component

```tsx
// src/components/pwa/AddToHomeScreen.tsx
import React from 'react';
import { IonToast } from '@ionic/react';
import { useAddToHomeScreen } from '@/hooks/useAddToHomeScreen';

export const AddToHomeScreen: React.FC = () => {
  const { showPrompt, install, dismiss } = useAddToHomeScreen();

  return (
    <IonToast
      isOpen={showPrompt}
      message="Install this app for a better experience"
      position="bottom"
      buttons={[
        {
          text: 'Install',
          role: 'confirm',
          handler: install,
        },
        {
          text: 'Later',
          role: 'cancel',
          handler: dismiss,
        },
      ]}
      onDidDismiss={dismiss}
    />
  );
};
```

## Update Prompt Component

```tsx
// src/components/pwa/UpdatePrompt.tsx
import React, { useState, useEffect } from 'react';
import { IonAlert } from '@ionic/react';

export const UpdatePrompt: React.FC = () => {
  const [showUpdate, setShowUpdate] = useState(false);

  useEffect(() => {
    const handler = () => setShowUpdate(true);
    window.addEventListener('sw-update-available', handler);
    return () => window.removeEventListener('sw-update-available', handler);
  }, []);

  const handleUpdate = () => {
    const updateSW = (window as any).updateSW;
    if (updateSW) {
      updateSW(true);
    }
  };

  return (
    <IonAlert
      isOpen={showUpdate}
      header="Update Available"
      message="A new version is available. Update now for the latest features."
      buttons={[
        {
          text: 'Later',
          role: 'cancel',
          handler: () => setShowUpdate(false),
        },
        {
          text: 'Update',
          handler: handleUpdate,
        },
      ]}
      onDidDismiss={() => setShowUpdate(false)}
    />
  );
};
```

## Network Status Hook

```typescript
// src/hooks/useNetworkStatus.ts
import { useState, useEffect } from 'react';

export function useNetworkStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return { isOnline };
}
```

## Offline Indicator Component

```tsx
// src/components/pwa/OfflineIndicator.tsx
import React, { useState, useEffect } from 'react';
import { IonChip, IonIcon, IonLabel } from '@ionic/react';
import { cloudOffline } from 'ionicons/icons';
import { useNetworkStatus } from '@/hooks/useNetworkStatus';
import { db } from '@/lib/offline/db';

export const OfflineIndicator: React.FC = () => {
  const { isOnline } = useNetworkStatus();
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
    <IonChip color="warning" className="offline-chip">
      <IonIcon icon={cloudOffline} />
      <IonLabel>
        Offline{pendingCount > 0 ? ` (${pendingCount} pending)` : ''}
      </IonLabel>
    </IonChip>
  );
};
```

## Pull to Refresh Page

```tsx
// src/pages/Home.tsx
import React, { useState, useEffect } from 'react';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonRefresher,
  IonRefresherContent,
  IonList,
  IonItem,
  IonLabel,
  IonSkeletonText,
  RefresherEventDetail,
} from '@ionic/react';

interface Item {
  id: string;
  name: string;
}

export const Home: React.FC = () => {
  const [loading, setLoading] = useState(true);
  const [items, setItems] = useState<Item[]>([]);

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    setLoading(true);
    try {
      // Load from local DB first, then sync
      // const data = await db.items.toArray();
      // setItems(data);
    } finally {
      setLoading(false);
    }
  };

  const handleRefresh = async (event: CustomEvent<RefresherEventDetail>) => {
    await loadData();
    event.detail.complete();
  };

  return (
    <IonPage>
      <IonHeader>
        <IonToolbar>
          <IonTitle>Home</IonTitle>
        </IonToolbar>
      </IonHeader>

      <IonContent fullscreen>
        <IonRefresher slot="fixed" onIonRefresh={handleRefresh}>
          <IonRefresherContent
            pullingIcon="chevron-down-circle-outline"
            pullingText="Pull to refresh"
            refreshingSpinner="circles"
            refreshingText="Refreshing..."
          />
        </IonRefresher>

        {loading ? (
          <IonList>
            {[1, 2, 3, 4, 5].map((i) => (
              <IonItem key={i}>
                <IonSkeletonText animated style={{ width: '60%' }} />
              </IonItem>
            ))}
          </IonList>
        ) : (
          <IonList>
            {items.map((item) => (
              <IonItem key={item.id}>
                <IonLabel>{item.name}</IonLabel>
              </IonItem>
            ))}
          </IonList>
        )}
      </IonContent>
    </IonPage>
  );
};
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
    super('IonicReactPWADatabase');

    this.version(1).stores({
      syncQueue: '++id, table, recordId, createdAt',
    });
  }
}

export const db = new AppDatabase();
```

## App.tsx with PWA Components

```tsx
// src/App.tsx
import React from 'react';
import { IonApp, IonRouterOutlet, setupIonicReact } from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { Route, Redirect } from 'react-router-dom';

import { Home } from './pages/Home';
import { AddToHomeScreen } from './components/pwa/AddToHomeScreen';
import { UpdatePrompt } from './components/pwa/UpdatePrompt';
import { OfflineIndicator } from './components/pwa/OfflineIndicator';

setupIonicReact();

const App: React.FC = () => {
  return (
    <IonApp>
      <OfflineIndicator />
      <IonReactRouter>
        <IonRouterOutlet>
          <Route exact path="/home" component={Home} />
          <Route exact path="/">
            <Redirect to="/home" />
          </Route>
        </IonRouterOutlet>
      </IonReactRouter>
      <AddToHomeScreen />
      <UpdatePrompt />
    </IonApp>
  );
};

export default App;
```

## CSS Variables for Safe Areas

```css
/* src/theme/variables.css - add to existing */

:root {
  --ion-safe-area-top: env(safe-area-inset-top);
  --ion-safe-area-bottom: env(safe-area-inset-bottom);
  --ion-safe-area-left: env(safe-area-inset-left);
  --ion-safe-area-right: env(safe-area-inset-right);
}

/* Offline indicator positioning */
.offline-chip {
  position: fixed;
  top: calc(var(--ion-safe-area-top, 0px) + 8px);
  left: 50%;
  transform: translateX(-50%);
  z-index: 9999;
}
```

## Setup Commands

```bash
# Create Ionic React project
bunx @ionic/cli start my-pwa blank --type=react --capacitor

cd my-pwa

# Install PWA dependencies
bun add -D vite-plugin-pwa workbox-window
bun add dexie zustand @tanstack/react-query axios axios-retry date-fns uuid
bun add -D @types/uuid

# Update vite.config.ts with PWA plugin

# Generate PWA icons
# Use https://realfavicongenerator.net

# Development
bun dev

# Build
bun run build

# Preview
bun run preview
```

## Key Rules

1. **Use Ionic's built-in hooks** - `useIonRouter`, `useIonLoading`, etc.
2. **Leverage IonRefresher** - Native-feeling pull-to-refresh
3. **Use IonSkeletonText** - Built-in skeleton loaders
4. **React Query for data** - Excellent cache management for offline
5. **Zustand for local state** - Simple, works offline
6. **Test with Capacitor** - Even for PWA, Capacitor improves iOS experience
7. **Use fullscreen on IonContent** - Proper safe area handling
