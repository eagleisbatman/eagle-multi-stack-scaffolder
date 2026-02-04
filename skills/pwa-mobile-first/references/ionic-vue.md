# Ionic Vue Reference (PWA)

## Research Queries
- "Ionic Vue PWA best practices 2025 2026"
- "Ionic Capacitor PWA hybrid approach"
- "Ionic Vue service worker Workbox"
- "Ionic Vue offline storage IndexedDB"
- "Ionic Vue iOS PWA limitations workarounds"

## Package Manager
**Bun** - Fast installs, fully compatible with Ionic CLI.

```bash
bunx @ionic/cli start my-pwa blank --type=vue --capacitor
```

## Why Ionic Vue for PWA

Choose Ionic Vue when:

1. **Vue Team**: Your team already knows Vue
2. **Native Path**: You might need Capacitor for native features later
3. **iOS + Android Design**: Automatic platform-adaptive styling
4. **Existing Ionic Knowledge**: Team has Ionic experience
5. **Component Library**: Rich set of mobile-optimized components

## Project Structure

```
my-pwa/
├── public/
│   ├── manifest.json             # PWA manifest
│   └── icons/                    # PWA icons
├── src/
│   ├── assets/
│   ├── components/
│   │   ├── ui/
│   │   │   ├── SkeletonCard.vue
│   │   │   └── EmptyState.vue
│   │   └── pwa/
│   │       ├── AddToHomeScreen.vue
│   │       ├── UpdatePrompt.vue
│   │       └── OfflineIndicator.vue
│   ├── composables/
│   │   ├── useOfflineSync.ts
│   │   ├── useNetworkStatus.ts
│   │   └── usePullToRefresh.ts
│   ├── router/
│   │   └── index.ts
│   ├── stores/
│   │   └── sync-queue.ts
│   ├── lib/
│   │   └── offline/
│   │       ├── db.ts
│   │       └── sync-manager.ts
│   ├── views/
│   │   ├── HomePage.vue
│   │   └── OfflinePage.vue
│   ├── theme/
│   │   └── variables.css
│   ├── App.vue
│   └── main.ts
├── capacitor.config.ts           # Capacitor config (even for PWA)
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
import vue from '@vitejs/plugin-vue';
import { VitePWA } from 'vite-plugin-pwa';
import path from 'path';

export default defineConfig({
  plugins: [
    vue(),
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
                maxAgeSeconds: 60 * 60 * 24 * 365, // 1 year
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
                maxAgeSeconds: 60 * 60 * 24, // 1 day
              },
              cacheableResponse: {
                statuses: [0, 200],
              },
            },
          },
        ],
      },
      manifest: {
        name: 'My Ionic PWA',
        short_name: 'MyPWA',
        description: 'A Progressive Web App built with Ionic Vue',
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
// src/main.ts
import { createApp } from 'vue';
import { createPinia } from 'pinia';
import { IonicVue } from '@ionic/vue';
import App from './App.vue';
import router from './router';

import '@ionic/vue/css/core.css';
import '@ionic/vue/css/normalize.css';
import '@ionic/vue/css/structure.css';
import '@ionic/vue/css/typography.css';
import '@ionic/vue/css/padding.css';
import '@ionic/vue/css/float-elements.css';
import '@ionic/vue/css/text-alignment.css';
import '@ionic/vue/css/text-transformation.css';
import '@ionic/vue/css/flex-utils.css';
import '@ionic/vue/css/display.css';
import './theme/variables.css';

// PWA Registration
import { registerSW } from 'virtual:pwa-register';

const app = createApp(App)
  .use(IonicVue)
  .use(createPinia())
  .use(router);

router.isReady().then(() => {
  app.mount('#app');
});

// Register service worker with update prompt
const updateSW = registerSW({
  onNeedRefresh() {
    // Emit event for UpdatePrompt component
    window.dispatchEvent(new CustomEvent('sw-update-available'));
  },
  onOfflineReady() {
    console.log('App ready to work offline');
  },
});

// Export for use in UpdatePrompt
(window as any).updateSW = updateSW;
```

## Add to Home Screen Component

```vue
<!-- src/components/pwa/AddToHomeScreen.vue -->
<template>
  <ion-toast
    :is-open="showPrompt"
    message="Install this app for a better experience"
    position="bottom"
    :buttons="toastButtons"
    @didDismiss="dismiss"
  />
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { IonToast } from '@ionic/vue';

const showPrompt = ref(false);
let deferredPrompt: BeforeInstallPromptEvent | null = null;

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

const toastButtons = computed(() => [
  {
    text: 'Install',
    role: 'confirm',
    handler: () => install(),
  },
  {
    text: 'Later',
    role: 'cancel',
    handler: () => dismiss(),
  },
]);

onMounted(() => {
  if (window.matchMedia('(display-mode: standalone)').matches) {
    return;
  }

  const dismissed = localStorage.getItem('a2hs-dismissed');
  if (dismissed) {
    const daysSince = (Date.now() - parseInt(dismissed)) / (1000 * 60 * 60 * 24);
    if (daysSince < 7) return;
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
```

## Update Prompt Component

```vue
<!-- src/components/pwa/UpdatePrompt.vue -->
<template>
  <ion-alert
    :is-open="showUpdate"
    header="Update Available"
    message="A new version is available. Update now for the latest features."
    :buttons="alertButtons"
    @didDismiss="showUpdate = false"
  />
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import { IonAlert } from '@ionic/vue';

const showUpdate = ref(false);

const alertButtons = computed(() => [
  {
    text: 'Later',
    role: 'cancel',
  },
  {
    text: 'Update',
    handler: () => {
      const updateSW = (window as any).updateSW;
      if (updateSW) {
        updateSW(true);
      }
    },
  },
]);

onMounted(() => {
  window.addEventListener('sw-update-available', () => {
    showUpdate.value = true;
  });
});
</script>
```

## Offline Indicator Component

```vue
<!-- src/components/pwa/OfflineIndicator.vue -->
<template>
  <ion-chip
    v-if="!isOnline"
    color="warning"
    class="offline-chip"
  >
    <ion-icon :icon="cloudOffline" />
    <ion-label>Offline{{ pendingCount > 0 ? ` (${pendingCount} pending)` : '' }}</ion-label>
  </ion-chip>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { IonChip, IonIcon, IonLabel } from '@ionic/vue';
import { cloudOffline } from 'ionicons/icons';
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

<style scoped>
.offline-chip {
  position: fixed;
  top: calc(var(--ion-safe-area-top, 0px) + 8px);
  left: 50%;
  transform: translateX(-50%);
  z-index: 9999;
}
</style>
```

## Pull to Refresh Component

```vue
<!-- src/views/HomePage.vue -->
<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Home</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-refresher slot="fixed" @ionRefresh="handleRefresh">
        <ion-refresher-content
          pulling-icon="chevron-down-circle-outline"
          pulling-text="Pull to refresh"
          refreshing-spinner="circles"
          refreshing-text="Refreshing..."
        />
      </ion-refresher>

      <!-- Content here -->
      <ion-list v-if="!loading">
        <ion-item v-for="item in items" :key="item.id">
          <ion-label>{{ item.name }}</ion-label>
        </ion-item>
      </ion-list>

      <!-- Skeleton loaders -->
      <ion-list v-else>
        <ion-item v-for="i in 5" :key="i">
          <ion-skeleton-text :animated="true" style="width: 60%" />
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonRefresher, IonRefresherContent,
  IonList, IonItem, IonLabel, IonSkeletonText,
} from '@ionic/vue';
import type { RefresherCustomEvent } from '@ionic/vue';

const loading = ref(true);
const items = ref<Array<{ id: string; name: string }>>([]);

onMounted(async () => {
  await loadData();
});

async function loadData() {
  loading.value = true;
  try {
    // Load from local DB first, then sync
    // items.value = await db.items.toArray();
  } finally {
    loading.value = false;
  }
}

async function handleRefresh(event: RefresherCustomEvent) {
  await loadData();
  event.target.complete();
}
</script>
```

## iOS Safe Area Variables

```css
/* src/theme/variables.css - add to existing */

:root {
  /* iOS safe area handling */
  --ion-safe-area-top: env(safe-area-inset-top);
  --ion-safe-area-bottom: env(safe-area-inset-bottom);
  --ion-safe-area-left: env(safe-area-inset-left);
  --ion-safe-area-right: env(safe-area-inset-right);
}

/* Ensure content respects safe areas */
ion-content {
  --padding-top: var(--ion-safe-area-top);
  --padding-bottom: var(--ion-safe-area-bottom);
}

/* Fixed elements need manual safe area handling */
.bottom-fixed {
  padding-bottom: calc(16px + var(--ion-safe-area-bottom));
}

/* Touch targets */
ion-button,
ion-item {
  --min-height: 44px;
}
```

## App.vue with PWA Components

```vue
<!-- src/App.vue -->
<template>
  <ion-app>
    <OfflineIndicator />
    <ion-router-outlet />
    <AddToHomeScreen />
    <UpdatePrompt />
  </ion-app>
</template>

<script setup lang="ts">
import { IonApp, IonRouterOutlet } from '@ionic/vue';
import AddToHomeScreen from '@/components/pwa/AddToHomeScreen.vue';
import UpdatePrompt from '@/components/pwa/UpdatePrompt.vue';
import OfflineIndicator from '@/components/pwa/OfflineIndicator.vue';
</script>
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
  // Add your tables here

  constructor() {
    super('IonicPWADatabase');

    this.version(1).stores({
      syncQueue: '++id, table, recordId, createdAt',
      // Add your tables here
    });
  }
}

export const db = new AppDatabase();
```

## Network Status Composable

```typescript
// src/composables/useNetworkStatus.ts
import { ref, onMounted, onUnmounted } from 'vue';

export function useNetworkStatus() {
  const isOnline = ref(navigator.onLine);

  const updateStatus = () => {
    isOnline.value = navigator.onLine;
  };

  onMounted(() => {
    window.addEventListener('online', updateStatus);
    window.addEventListener('offline', updateStatus);
  });

  onUnmounted(() => {
    window.removeEventListener('online', updateStatus);
    window.removeEventListener('offline', updateStatus);
  });

  return { isOnline };
}
```

## Setup Commands

```bash
# Create Ionic Vue project
bunx @ionic/cli start my-pwa blank --type=vue --capacitor

cd my-pwa

# Install PWA dependencies
bun add -D vite-plugin-pwa workbox-window
bun add dexie axios axios-retry date-fns uuid
bun add -D @types/uuid

# Update vite.config.ts with PWA plugin

# Generate PWA icons
# Use https://realfavicongenerator.net or similar

# Development
bun dev

# Build
bun run build

# Preview
bun run preview
```

## Key Rules

1. **Use Ionic's built-in components** - They handle safe areas automatically
2. **Leverage IonRefresher** - Native-feeling pull-to-refresh
3. **Use IonSkeletonText** - Built-in skeleton loaders
4. **Test with Capacitor** - Even for PWA, Capacitor improves iOS experience
5. **Respect platform conventions** - Ionic auto-adapts to iOS/Android styles
6. **Use ion-content's fullscreen** - Proper safe area handling
7. **PWA + Capacitor works** - You can deploy as PWA and wrap with Capacitor later
