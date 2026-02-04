# Quasar Framework Reference (PWA)

## Research Queries
- "Quasar PWA best practices 2025 2026"
- "Quasar service worker Workbox configuration"
- "Quasar offline first IndexedDB Dexie"
- "Quasar PWA iOS Safari issues workarounds"
- "Quasar Capacitor hybrid app migration"

## Package Manager
**Bun** - Fast installs, fully compatible with Quasar CLI.

```bash
bun create quasar my-pwa
# Select: Quasar v2, TypeScript, Composition API, Pinia, PWA mode
```

## Why Quasar for PWA

Quasar is the **recommended framework** for PWAs because:

1. **Built-in PWA Mode**: First-class PWA support, not an afterthought
2. **Workbox Integration**: Pre-configured service worker with sensible defaults
3. **Material + iOS Design**: Automatic platform-adaptive styling
4. **Offline-First Components**: Built-in loading states, pull-to-refresh
5. **SSR Support**: Can add SEO if needed later
6. **Capacitor Ready**: Easy path to native if requirements change

## Project Structure

```
my-pwa/
├── public/
│   ├── icons/                    # PWA icons (all sizes)
│   │   ├── icon-128x128.png
│   │   ├── icon-192x192.png
│   │   ├── icon-256x256.png
│   │   ├── icon-384x384.png
│   │   ├── icon-512x512.png
│   │   └── maskable-icon-512x512.png
│   └── favicon.ico
├── src/
│   ├── assets/                   # Static assets
│   ├── boot/                     # Boot files (plugins)
│   │   ├── offline.ts           # Dexie.js setup
│   │   ├── pwa.ts               # PWA utilities (A2HS, updates)
│   │   └── axios.ts             # HTTP client with offline queue
│   ├── components/
│   │   ├── ui/                  # Base UI components
│   │   │   ├── SkeletonCard.vue
│   │   │   ├── SkeletonList.vue
│   │   │   └── EmptyState.vue
│   │   ├── pwa/                 # PWA-specific components
│   │   │   ├── AddToHomeScreen.vue
│   │   │   ├── UpdatePrompt.vue
│   │   │   └── OfflineIndicator.vue
│   │   └── forms/               # Form components
│   │       └── OfflineForm.vue
│   ├── composables/             # Vue composables
│   │   ├── useOfflineSync.ts
│   │   ├── useNetworkStatus.ts
│   │   └── usePullToRefresh.ts
│   ├── layouts/
│   │   └── MainLayout.vue
│   ├── pages/
│   │   ├── IndexPage.vue
│   │   └── OfflinePage.vue      # Fallback for offline
│   ├── router/
│   │   └── routes.ts
│   ├── stores/                   # Pinia stores
│   │   ├── app.ts               # App state
│   │   └── sync-queue.ts        # Offline sync queue
│   ├── lib/
│   │   └── offline/
│   │       ├── db.ts            # Dexie database schema
│   │       ├── sync-manager.ts  # Sync logic
│   │       └── conflict-resolver.ts
│   └── css/
│       ├── app.scss
│       └── quasar.variables.scss
├── src-pwa/
│   ├── custom-service-worker.ts  # Extended Workbox config
│   ├── register-service-worker.ts
│   └── pwa-env.d.ts
├── quasar.config.ts              # Quasar configuration
├── package.json
└── tsconfig.json
```

## Essential Libraries

```bash
# Offline storage
bun add dexie dexie-observable

# State management (included by default)
# bun add pinia @pinia/nuxt

# HTTP client with retry
bun add axios axios-retry

# Utilities
bun add date-fns uuid
bun add -D @types/uuid
```

## Quasar Config for PWA

```typescript
// quasar.config.ts
import { configure } from 'quasar/wrappers';

export default configure((/* ctx */) => {
  return {
    boot: ['offline', 'pwa', 'axios'],

    css: ['app.scss'],

    extras: [
      'roboto-font',
      'material-icons',
    ],

    build: {
      target: {
        browser: ['es2022', 'chrome100', 'firefox100', 'safari15'],
      },
      vueRouterMode: 'history',
    },

    pwa: {
      workboxMode: 'InjectManifest', // For custom service worker
      injectPwaMetaTags: true,
      swFilename: 'sw.js',
      manifestFilename: 'manifest.json',
      useCredentialsForManifestTag: false,

      manifest: {
        name: 'My PWA App',
        short_name: 'MyPWA',
        description: 'A Progressive Web App',
        display: 'standalone',
        orientation: 'portrait-primary',
        background_color: '#ffffff',
        theme_color: '#1976d2',
        start_url: '/',
        icons: [
          {
            src: 'icons/icon-128x128.png',
            sizes: '128x128',
            type: 'image/png',
          },
          {
            src: 'icons/icon-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: 'icons/icon-256x256.png',
            sizes: '256x256',
            type: 'image/png',
          },
          {
            src: 'icons/icon-384x384.png',
            sizes: '384x384',
            type: 'image/png',
          },
          {
            src: 'icons/icon-512x512.png',
            sizes: '512x512',
            type: 'image/png',
          },
          {
            src: 'icons/maskable-icon-512x512.png',
            sizes: '512x512',
            type: 'image/png',
            purpose: 'maskable',
          },
        ],
      },

      // iOS meta tags
      metaVariables: {
        appleMobileWebAppCapable: 'yes',
        appleMobileWebAppStatusBarStyle: 'default',
        appleTouchIcon120: 'icons/apple-icon-120x120.png',
        appleTouchIcon180: 'icons/apple-icon-180x180.png',
        appleTouchIcon152: 'icons/apple-icon-152x152.png',
        appleTouchIcon167: 'icons/apple-icon-167x167.png',
        msapplicationTileImage: 'icons/ms-icon-144x144.png',
        msapplicationTileColor: '#1976d2',
      },
    },
  };
});
```

## Custom Service Worker

```typescript
// src-pwa/custom-service-worker.ts
import { precacheAndRoute, cleanupOutdatedCaches } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { StaleWhileRevalidate, CacheFirst, NetworkFirst } from 'workbox-strategies';
import { ExpirationPlugin } from 'workbox-expiration';
import { CacheableResponsePlugin } from 'workbox-cacheable-response';
import { BackgroundSyncPlugin } from 'workbox-background-sync';

declare const self: ServiceWorkerGlobalScope & typeof globalThis;

// Precache app shell
precacheAndRoute(self.__WB_MANIFEST);
cleanupOutdatedCaches();

// Cache Google Fonts
registerRoute(
  ({ url }) => url.origin === 'https://fonts.googleapis.com' ||
               url.origin === 'https://fonts.gstatic.com',
  new CacheFirst({
    cacheName: 'google-fonts',
    plugins: [
      new CacheableResponsePlugin({ statuses: [0, 200] }),
      new ExpirationPlugin({ maxEntries: 30, maxAgeSeconds: 60 * 60 * 24 * 365 }),
    ],
  })
);

// Cache images
registerRoute(
  ({ request }) => request.destination === 'image',
  new CacheFirst({
    cacheName: 'images',
    plugins: [
      new CacheableResponsePlugin({ statuses: [0, 200] }),
      new ExpirationPlugin({ maxEntries: 100, maxAgeSeconds: 60 * 60 * 24 * 30 }),
    ],
  })
);

// API calls - Network First with offline fallback
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkFirst({
    cacheName: 'api-cache',
    networkTimeoutSeconds: 10,
    plugins: [
      new CacheableResponsePlugin({ statuses: [0, 200] }),
      new ExpirationPlugin({ maxEntries: 50, maxAgeSeconds: 60 * 60 * 24 }),
    ],
  })
);

// Background sync for POST/PUT/DELETE requests
const bgSyncPlugin = new BackgroundSyncPlugin('api-queue', {
  maxRetentionTime: 24 * 60, // 24 hours
});

registerRoute(
  ({ url, request }) =>
    url.pathname.startsWith('/api/') &&
    ['POST', 'PUT', 'DELETE', 'PATCH'].includes(request.method),
  new NetworkFirst({
    cacheName: 'api-mutations',
    plugins: [bgSyncPlugin],
  }),
  'POST'
);

// Offline fallback page
const OFFLINE_PAGE = '/offline';
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('offline-fallback').then((cache) => cache.add(OFFLINE_PAGE))
  );
});

self.addEventListener('fetch', (event) => {
  if (event.request.mode === 'navigate') {
    event.respondWith(
      fetch(event.request).catch(() =>
        caches.match(OFFLINE_PAGE) as Promise<Response>
      )
    );
  }
});

// Listen for skip waiting message
self.addEventListener('message', (event) => {
  if (event.data && event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});
```

## Dexie.js Database Setup

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

// Example: Inspection app
export interface Inspection extends SyncableRecord {
  title: string;
  location: string;
  status: 'draft' | 'pending' | 'completed';
  notes: string;
  photos: string[]; // base64 or blob URLs
  createdAt: number;
  updatedAt: number;
}

export class AppDatabase extends Dexie {
  inspections!: Table<Inspection, string>;
  syncQueue!: Table<SyncQueueItem, number>;

  constructor() {
    super('MyPWADatabase');

    this.version(1).stores({
      inspections: 'id, status, _synced, _localUpdatedAt',
      syncQueue: '++id, table, recordId, createdAt',
    });
  }
}

export const db = new AppDatabase();
```

## Sync Manager

```typescript
// src/lib/offline/sync-manager.ts
import { db, SyncQueueItem, SyncableRecord } from './db';
import { api } from '@/boot/axios';

class SyncManager {
  private isSyncing = false;
  private onlineHandler: () => void;

  constructor() {
    this.onlineHandler = () => this.processQueue();
    window.addEventListener('online', this.onlineHandler);
  }

  async queueOperation<T extends SyncableRecord>(
    operation: 'create' | 'update' | 'delete',
    table: string,
    record: T
  ): Promise<void> {
    // Save to local DB first
    const localRecord = {
      ...record,
      _synced: false,
      _localUpdatedAt: Date.now(),
    };

    const tableRef = db.table(table);

    if (operation === 'delete') {
      await tableRef.update(record.id, { _deleted: true, _synced: false });
    } else {
      await tableRef.put(localRecord);
    }

    // Queue for sync
    await db.syncQueue.add({
      operation,
      table,
      recordId: record.id,
      data: record as Record<string, unknown>,
      createdAt: Date.now(),
      retryCount: 0,
    });

    // Try to sync immediately if online
    if (navigator.onLine) {
      this.processQueue();
    }
  }

  async processQueue(): Promise<void> {
    if (this.isSyncing || !navigator.onLine) return;

    this.isSyncing = true;

    try {
      const items = await db.syncQueue.orderBy('createdAt').toArray();

      for (const item of items) {
        try {
          await this.syncItem(item);
          await db.syncQueue.delete(item.id!);

          // Mark record as synced
          const tableRef = db.table(item.table);
          if (item.operation !== 'delete') {
            await tableRef.update(item.recordId, {
              _synced: true,
              _syncedAt: Date.now(),
            });
          } else {
            await tableRef.delete(item.recordId);
          }
        } catch (error) {
          console.error(`Sync failed for ${item.table}/${item.recordId}:`, error);

          // Increment retry count
          await db.syncQueue.update(item.id!, {
            retryCount: item.retryCount + 1,
          });

          // Remove after max retries
          if (item.retryCount >= 5) {
            console.error(`Max retries reached for ${item.table}/${item.recordId}`);
            // Optionally: move to dead letter queue
          }
        }
      }
    } finally {
      this.isSyncing = false;
    }
  }

  private async syncItem(item: SyncQueueItem): Promise<void> {
    const endpoint = `/api/${item.table}`;

    switch (item.operation) {
      case 'create':
        await api.post(endpoint, item.data);
        break;
      case 'update':
        await api.put(`${endpoint}/${item.recordId}`, item.data);
        break;
      case 'delete':
        await api.delete(`${endpoint}/${item.recordId}`);
        break;
    }
  }

  async getPendingCount(): Promise<number> {
    return db.syncQueue.count();
  }

  destroy(): void {
    window.removeEventListener('online', this.onlineHandler);
  }
}

export const syncManager = new SyncManager();
```

## Network Status Composable

```typescript
// src/composables/useNetworkStatus.ts
import { ref, onMounted, onUnmounted } from 'vue';

export function useNetworkStatus() {
  const isOnline = ref(navigator.onLine);
  const connectionType = ref<string | null>(null);

  const updateStatus = () => {
    isOnline.value = navigator.onLine;

    // @ts-ignore - Navigator connection API
    if (navigator.connection) {
      // @ts-ignore
      connectionType.value = navigator.connection.effectiveType;
    }
  };

  onMounted(() => {
    window.addEventListener('online', updateStatus);
    window.addEventListener('offline', updateStatus);

    // @ts-ignore
    if (navigator.connection) {
      // @ts-ignore
      navigator.connection.addEventListener('change', updateStatus);
    }
  });

  onUnmounted(() => {
    window.removeEventListener('online', updateStatus);
    window.removeEventListener('offline', updateStatus);

    // @ts-ignore
    if (navigator.connection) {
      // @ts-ignore
      navigator.connection.removeEventListener('change', updateStatus);
    }
  });

  return {
    isOnline,
    connectionType,
  };
}
```

## Add to Home Screen Component

```vue
<!-- src/components/pwa/AddToHomeScreen.vue -->
<template>
  <q-banner
    v-if="showPrompt"
    class="bg-primary text-white a2hs-banner"
    dense
  >
    <template #avatar>
      <q-icon name="install_mobile" size="24px" />
    </template>

    <div class="text-body2">
      Install this app for a better experience
    </div>

    <template #action>
      <q-btn
        flat
        label="Install"
        @click="install"
      />
      <q-btn
        flat
        icon="close"
        @click="dismiss"
      />
    </template>
  </q-banner>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const showPrompt = ref(false);
let deferredPrompt: BeforeInstallPromptEvent | null = null;

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

onMounted(() => {
  // Check if already installed
  if (window.matchMedia('(display-mode: standalone)').matches) {
    return;
  }

  // Check if dismissed recently
  const dismissed = localStorage.getItem('a2hs-dismissed');
  if (dismissed) {
    const dismissedAt = parseInt(dismissed, 10);
    const daysSinceDismissed = (Date.now() - dismissedAt) / (1000 * 60 * 60 * 24);
    if (daysSinceDismissed < 7) {
      return;
    }
  }

  window.addEventListener('beforeinstallprompt', (e: Event) => {
    e.preventDefault();
    deferredPrompt = e as BeforeInstallPromptEvent;
    showPrompt.value = true;
  });
});

async function install() {
  if (!deferredPrompt) return;

  deferredPrompt.prompt();
  const { outcome } = await deferredPrompt.userChoice;

  if (outcome === 'accepted') {
    showPrompt.value = false;
  }

  deferredPrompt = null;
}

function dismiss() {
  showPrompt.value = false;
  localStorage.setItem('a2hs-dismissed', Date.now().toString());
}
</script>

<style scoped lang="scss">
.a2hs-banner {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 9999;
  padding-bottom: env(safe-area-inset-bottom);
}
</style>
```

## Update Prompt Component

```vue
<!-- src/components/pwa/UpdatePrompt.vue -->
<template>
  <q-dialog v-model="showUpdate" persistent>
    <q-card style="min-width: 280px">
      <q-card-section class="row items-center">
        <q-icon name="system_update" size="32px" color="primary" class="q-mr-md" />
        <div class="text-h6">Update Available</div>
      </q-card-section>

      <q-card-section>
        A new version of the app is available. Update now for the latest features and fixes.
      </q-card-section>

      <q-card-actions align="right">
        <q-btn flat label="Later" @click="showUpdate = false" />
        <q-btn color="primary" label="Update Now" @click="updateApp" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const showUpdate = ref(false);
let registration: ServiceWorkerRegistration | null = null;

onMounted(() => {
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.ready.then((reg) => {
      registration = reg;

      reg.addEventListener('updatefound', () => {
        const newWorker = reg.installing;
        if (!newWorker) return;

        newWorker.addEventListener('statechange', () => {
          if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
            showUpdate.value = true;
          }
        });
      });
    });

    // Listen for controller change (after skipWaiting)
    let refreshing = false;
    navigator.serviceWorker.addEventListener('controllerchange', () => {
      if (!refreshing) {
        refreshing = true;
        window.location.reload();
      }
    });
  }
});

function updateApp() {
  if (registration?.waiting) {
    registration.waiting.postMessage({ type: 'SKIP_WAITING' });
  }
}
</script>
```

## Offline Indicator Component

```vue
<!-- src/components/pwa/OfflineIndicator.vue -->
<template>
  <Transition name="slide-down">
    <q-banner
      v-if="!isOnline"
      class="bg-warning text-dark offline-banner"
      dense
    >
      <template #avatar>
        <q-icon name="cloud_off" />
      </template>
      <span>You're offline. Changes will sync when you're back online.</span>
      <span v-if="pendingCount > 0" class="q-ml-sm">
        ({{ pendingCount }} pending)
      </span>
    </q-banner>
  </Transition>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { useNetworkStatus } from '@/composables/useNetworkStatus';
import { db } from '@/lib/offline/db';

const { isOnline } = useNetworkStatus();
const pendingCount = ref(0);

let interval: ReturnType<typeof setInterval>;

onMounted(() => {
  updatePendingCount();
  interval = setInterval(updatePendingCount, 5000);
});

onUnmounted(() => {
  clearInterval(interval);
});

async function updatePendingCount() {
  pendingCount.value = await db.syncQueue.count();
}
</script>

<style scoped lang="scss">
.offline-banner {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 9999;
  padding-top: env(safe-area-inset-top);
}

.slide-down-enter-active,
.slide-down-leave-active {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.slide-down-enter-from,
.slide-down-leave-to {
  transform: translateY(-100%);
  opacity: 0;
}
</style>
```

## Main Layout with PWA Components

```vue
<!-- src/layouts/MainLayout.vue -->
<template>
  <q-layout view="hHh lpR fFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          @click="toggleLeftDrawer"
        />
        <q-toolbar-title>My PWA</q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" side="left" bordered>
      <!-- Navigation -->
    </q-drawer>

    <q-page-container>
      <OfflineIndicator />
      <router-view />
    </q-page-container>

    <AddToHomeScreen />
    <UpdatePrompt />
  </q-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import AddToHomeScreen from '@/components/pwa/AddToHomeScreen.vue';
import UpdatePrompt from '@/components/pwa/UpdatePrompt.vue';
import OfflineIndicator from '@/components/pwa/OfflineIndicator.vue';

const leftDrawerOpen = ref(false);

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value;
}
</script>
```

## Setup Commands

```bash
# Create Quasar project with PWA
bun create quasar my-pwa
# Selections:
# - Quasar v2
# - TypeScript
# - Composition API
# - Pinia
# - SCSS
# - ESLint + Prettier
# - PWA mode

cd my-pwa

# Install additional dependencies
bun add dexie axios axios-retry date-fns uuid
bun add -D @types/uuid

# Generate PWA icons (use realfavicongenerator.net or similar)
# Place icons in public/icons/

# Development
bun dev

# Build for production
bun run build

# Preview production build
bun run preview
```

## Key Rules

1. **Always use InjectManifest mode** for complex service workers
2. **Test on iOS Safari** - most PWA issues appear there first
3. **Use `env(safe-area-inset-*)` everywhere** - handles notches and home indicators
4. **Never block on network** - show cached data immediately, update in background
5. **Always show sync status** - users must know when data is pending
6. **Use Background Sync API** - let the browser handle retry logic
7. **Implement proper error boundaries** - offline states shouldn't crash the app
8. **Cache API responses aggressively** - but show stale-while-revalidate UI patterns
