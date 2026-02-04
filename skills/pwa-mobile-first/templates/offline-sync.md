# Offline Data Sync with Dexie.js

## Overview

Dexie.js is a wrapper around IndexedDB that provides a much better developer experience. Combined with Background Sync, it enables robust offline-first data management.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        UI Layer                              │
│              (React Query / Pinia / Zustand)                 │
├─────────────────────────────────────────────────────────────┤
│                     Sync Manager                             │
│     (Orchestrates local storage and server sync)             │
├─────────────────────────────────────────────────────────────┤
│                      Dexie.js                                │
│               (IndexedDB abstraction)                        │
├─────────────────────────────────────────────────────────────┤
│                    IndexedDB                                 │
│              (Browser storage, ~50MB+)                       │
├─────────────────────────────────────────────────────────────┤
│                   Service Worker                             │
│         (Background Sync API, Network Detection)             │
└─────────────────────────────────────────────────────────────┘
```

## Dexie.js Database Setup

```typescript
// lib/offline/db.ts
import Dexie, { Table } from 'dexie';

// ===========================================
// BASE INTERFACES
// ===========================================

/**
 * Base interface for syncable records
 * All entities that sync with server should extend this
 */
export interface SyncableRecord {
  id: string;                  // UUID, generated client-side
  _synced: boolean;           // Has this record been synced?
  _syncedAt?: number;         // Timestamp of last successful sync
  _localUpdatedAt: number;    // Timestamp of last local update
  _deleted?: boolean;         // Soft delete flag for sync
  _version?: number;          // Version for conflict resolution
}

/**
 * Sync queue item for pending operations
 */
export interface SyncQueueItem {
  id?: number;                // Auto-incremented
  operation: 'create' | 'update' | 'delete';
  table: string;              // Table name
  recordId: string;           // Record ID
  data: Record<string, unknown>;
  createdAt: number;
  retryCount: number;
  lastError?: string;
}

/**
 * Sync metadata
 */
export interface SyncMeta {
  key: string;
  value: string | number;
}

// ===========================================
// DOMAIN MODELS (Example: Task App)
// ===========================================

export interface Task extends SyncableRecord {
  title: string;
  description?: string;
  status: 'todo' | 'in_progress' | 'done';
  dueDate?: number;
  priority: 'low' | 'medium' | 'high';
  tags: string[];
  createdAt: number;
  updatedAt: number;
}

export interface Project extends SyncableRecord {
  name: string;
  color: string;
  createdAt: number;
}

// ===========================================
// DATABASE CLASS
// ===========================================

export class AppDatabase extends Dexie {
  // Tables
  tasks!: Table<Task, string>;
  projects!: Table<Project, string>;
  syncQueue!: Table<SyncQueueItem, number>;
  syncMeta!: Table<SyncMeta, string>;

  constructor() {
    super('MyAppDatabase');

    // Define schema
    this.version(1).stores({
      // Primary key is 'id', indexes after comma
      tasks: 'id, status, priority, dueDate, _synced, _localUpdatedAt, _deleted',
      projects: 'id, name, _synced, _localUpdatedAt',
      syncQueue: '++id, table, recordId, createdAt, retryCount',
      syncMeta: 'key',
    });

    // Hooks for automatic timestamps
    this.tasks.hook('creating', (primKey, obj) => {
      obj._localUpdatedAt = Date.now();
      obj._synced = false;
    });

    this.tasks.hook('updating', (modifications) => {
      return {
        ...modifications,
        _localUpdatedAt: Date.now(),
        _synced: false,
      };
    });
  }
}

// Singleton instance
export const db = new AppDatabase();

// ===========================================
// UTILITY FUNCTIONS
// ===========================================

/**
 * Generate a UUID for new records
 */
export function generateId(): string {
  return crypto.randomUUID();
}

/**
 * Get all unsynced records from a table
 */
export async function getUnsyncedRecords<T extends SyncableRecord>(
  table: Table<T, string>
): Promise<T[]> {
  return table.where('_synced').equals(0).toArray();
}

/**
 * Mark record as synced
 */
export async function markSynced<T extends SyncableRecord>(
  table: Table<T, string>,
  id: string
): Promise<void> {
  await table.update(id, {
    _synced: true,
    _syncedAt: Date.now(),
  } as Partial<T>);
}
```

## Sync Manager

```typescript
// lib/offline/sync-manager.ts
import { db, SyncQueueItem, SyncableRecord, generateId } from './db';
import { api } from '@/lib/api';

type SyncOperation = 'create' | 'update' | 'delete';

interface SyncResult {
  success: boolean;
  synced: number;
  failed: number;
  errors: string[];
}

class SyncManager {
  private isSyncing = false;
  private listeners = new Set<(status: SyncStatus) => void>();

  // ===========================================
  // CRUD OPERATIONS (Always local-first)
  // ===========================================

  /**
   * Create a new record (offline-safe)
   */
  async create<T extends SyncableRecord>(
    tableName: string,
    data: Omit<T, keyof SyncableRecord>
  ): Promise<T> {
    const now = Date.now();
    const record: T = {
      ...data,
      id: generateId(),
      _synced: false,
      _localUpdatedAt: now,
      _version: 1,
    } as T;

    // Save locally
    const table = db.table(tableName);
    await table.add(record);

    // Queue for sync
    await this.queueOperation('create', tableName, record);

    // Try to sync immediately if online
    if (navigator.onLine) {
      this.processQueue();
    }

    return record;
  }

  /**
   * Update an existing record (offline-safe)
   */
  async update<T extends SyncableRecord>(
    tableName: string,
    id: string,
    changes: Partial<Omit<T, keyof SyncableRecord>>
  ): Promise<T | undefined> {
    const table = db.table<T, string>(tableName);
    const existing = await table.get(id);

    if (!existing) {
      throw new Error(`Record ${id} not found in ${tableName}`);
    }

    const updated: T = {
      ...existing,
      ...changes,
      _synced: false,
      _localUpdatedAt: Date.now(),
      _version: (existing._version || 0) + 1,
    };

    await table.put(updated);
    await this.queueOperation('update', tableName, updated);

    if (navigator.onLine) {
      this.processQueue();
    }

    return updated;
  }

  /**
   * Delete a record (soft delete for sync, then hard delete)
   */
  async delete(tableName: string, id: string): Promise<void> {
    const table = db.table(tableName);
    const existing = await table.get(id);

    if (!existing) return;

    // Soft delete locally
    await table.update(id, {
      _deleted: true,
      _synced: false,
      _localUpdatedAt: Date.now(),
    });

    // Queue for sync
    await this.queueOperation('delete', tableName, { id } as any);

    if (navigator.onLine) {
      this.processQueue();
    }
  }

  // ===========================================
  // SYNC QUEUE
  // ===========================================

  /**
   * Add operation to sync queue
   */
  private async queueOperation(
    operation: SyncOperation,
    table: string,
    data: Record<string, unknown>
  ): Promise<void> {
    // Check for existing queue item for same record
    const existing = await db.syncQueue
      .where(['table', 'recordId'])
      .equals([table, data.id as string])
      .first();

    if (existing) {
      // Update existing queue item
      await db.syncQueue.update(existing.id!, {
        operation: this.mergeOperations(existing.operation, operation),
        data,
        createdAt: Date.now(),
        retryCount: 0,
      });
    } else {
      // Add new queue item
      await db.syncQueue.add({
        operation,
        table,
        recordId: data.id as string,
        data,
        createdAt: Date.now(),
        retryCount: 0,
      });
    }

    this.notifyListeners({ pending: await this.getPendingCount() });
  }

  /**
   * Merge operations (create + update = create, create + delete = nothing)
   */
  private mergeOperations(
    existing: SyncOperation,
    incoming: SyncOperation
  ): SyncOperation {
    if (existing === 'create' && incoming === 'update') return 'create';
    if (existing === 'create' && incoming === 'delete') return 'delete';
    return incoming;
  }

  // ===========================================
  // SYNC PROCESSING
  // ===========================================

  /**
   * Process all pending sync operations
   */
  async processQueue(): Promise<SyncResult> {
    if (this.isSyncing || !navigator.onLine) {
      return { success: false, synced: 0, failed: 0, errors: ['Not ready'] };
    }

    this.isSyncing = true;
    this.notifyListeners({ syncing: true });

    const result: SyncResult = {
      success: true,
      synced: 0,
      failed: 0,
      errors: [],
    };

    try {
      const items = await db.syncQueue.orderBy('createdAt').toArray();

      for (const item of items) {
        try {
          await this.syncItem(item);
          await db.syncQueue.delete(item.id!);

          // Mark record as synced
          if (item.operation !== 'delete') {
            const table = db.table(item.table);
            await table.update(item.recordId, {
              _synced: true,
              _syncedAt: Date.now(),
            });
          } else {
            // Hard delete after successful sync
            const table = db.table(item.table);
            await table.delete(item.recordId);
          }

          result.synced++;
        } catch (error) {
          result.failed++;
          result.errors.push(`${item.table}/${item.recordId}: ${error}`);

          // Update retry count
          await db.syncQueue.update(item.id!, {
            retryCount: item.retryCount + 1,
            lastError: String(error),
          });

          // Remove after max retries
          if (item.retryCount >= 5) {
            console.error(`Max retries for ${item.table}/${item.recordId}`);
            // Optionally: move to dead letter queue
            await db.syncQueue.delete(item.id!);
          }
        }
      }

      result.success = result.failed === 0;
    } finally {
      this.isSyncing = false;
      this.notifyListeners({
        syncing: false,
        pending: await this.getPendingCount(),
      });
    }

    return result;
  }

  /**
   * Sync a single item
   */
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

  // ===========================================
  // FULL SYNC (Server → Client)
  // ===========================================

  /**
   * Pull all data from server (full refresh)
   */
  async fullSync(tableName: string): Promise<void> {
    const lastSync = await this.getLastSyncTime(tableName);
    const endpoint = `/api/${tableName}?since=${lastSync}`;

    const response = await api.get(endpoint);
    const serverRecords = response.data;

    const table = db.table(tableName);

    await db.transaction('rw', table, async () => {
      for (const serverRecord of serverRecords) {
        const local = await table.get(serverRecord.id);

        if (!local) {
          // New from server
          await table.add({
            ...serverRecord,
            _synced: true,
            _syncedAt: Date.now(),
            _localUpdatedAt: serverRecord.updatedAt,
          });
        } else if (!local._synced) {
          // Local changes exist - conflict resolution
          await this.resolveConflict(table, local, serverRecord);
        } else {
          // Server is newer
          await table.put({
            ...serverRecord,
            _synced: true,
            _syncedAt: Date.now(),
            _localUpdatedAt: serverRecord.updatedAt,
          });
        }
      }
    });

    await this.setLastSyncTime(tableName, Date.now());
  }

  /**
   * Resolve conflict between local and server versions
   */
  private async resolveConflict<T extends SyncableRecord>(
    table: Table<T, string>,
    local: T,
    server: any
  ): Promise<void> {
    // Strategy: Last-Write-Wins based on _version or timestamp
    // You can implement more sophisticated conflict resolution here

    const localTime = local._localUpdatedAt || 0;
    const serverTime = server.updatedAt || 0;

    if (localTime > serverTime) {
      // Keep local, re-queue for sync
      await this.queueOperation('update', table.name, local as any);
    } else {
      // Server wins, overwrite local
      await table.put({
        ...server,
        _synced: true,
        _syncedAt: Date.now(),
        _localUpdatedAt: server.updatedAt,
      });
    }
  }

  // ===========================================
  // METADATA & STATUS
  // ===========================================

  async getPendingCount(): Promise<number> {
    return db.syncQueue.count();
  }

  async getLastSyncTime(table: string): Promise<number> {
    const meta = await db.syncMeta.get(`lastSync_${table}`);
    return meta ? Number(meta.value) : 0;
  }

  async setLastSyncTime(table: string, time: number): Promise<void> {
    await db.syncMeta.put({ key: `lastSync_${table}`, value: time });
  }

  // ===========================================
  // EVENT LISTENERS
  // ===========================================

  subscribe(listener: (status: SyncStatus) => void): () => void {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  private notifyListeners(status: Partial<SyncStatus>): void {
    this.listeners.forEach((listener) =>
      listener({
        syncing: false,
        pending: 0,
        lastSync: Date.now(),
        ...status,
      })
    );
  }
}

export interface SyncStatus {
  syncing: boolean;
  pending: number;
  lastSync: number;
}

// Singleton
export const syncManager = new SyncManager();

// Auto-sync when coming online
if (typeof window !== 'undefined') {
  window.addEventListener('online', () => {
    syncManager.processQueue();
  });
}
```

## React Hooks

```typescript
// hooks/useOfflineSync.ts
import { useState, useEffect, useCallback } from 'react';
import { syncManager, SyncStatus } from '@/lib/offline/sync-manager';
import { db } from '@/lib/offline/db';
import { useLiveQuery } from 'dexie-react-hooks';

/**
 * Hook for sync status
 */
export function useSyncStatus() {
  const [status, setStatus] = useState<SyncStatus>({
    syncing: false,
    pending: 0,
    lastSync: 0,
  });

  useEffect(() => {
    return syncManager.subscribe(setStatus);
  }, []);

  const sync = useCallback(async () => {
    return syncManager.processQueue();
  }, []);

  return { ...status, sync };
}

/**
 * Hook for network status
 */
export function useNetworkStatus() {
  const [isOnline, setIsOnline] = useState(
    typeof navigator !== 'undefined' ? navigator.onLine : true
  );

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

  return isOnline;
}

/**
 * Hook for live query with Dexie
 */
export function useTasks(filter?: { status?: string }) {
  return useLiveQuery(async () => {
    let query = db.tasks.where('_deleted').notEqual(true);

    if (filter?.status) {
      query = db.tasks.where('status').equals(filter.status);
    }

    return query.toArray();
  }, [filter?.status]);
}

/**
 * Hook for single record
 */
export function useTask(id: string) {
  return useLiveQuery(() => db.tasks.get(id), [id]);
}
```

## Vue Composables

```typescript
// composables/useOfflineSync.ts
import { ref, onMounted, onUnmounted } from 'vue';
import { syncManager, SyncStatus } from '@/lib/offline/sync-manager';
import { db } from '@/lib/offline/db';
import { liveQuery } from 'dexie';

export function useSyncStatus() {
  const status = ref<SyncStatus>({
    syncing: false,
    pending: 0,
    lastSync: 0,
  });

  onMounted(() => {
    const unsubscribe = syncManager.subscribe((newStatus) => {
      status.value = newStatus;
    });

    onUnmounted(unsubscribe);
  });

  const sync = () => syncManager.processQueue();

  return { status, sync };
}

export function useNetworkStatus() {
  const isOnline = ref(navigator.onLine);

  onMounted(() => {
    const handleOnline = () => (isOnline.value = true);
    const handleOffline = () => (isOnline.value = false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    onUnmounted(() => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    });
  });

  return { isOnline };
}

export function useTasks() {
  const tasks = ref<Task[]>([]);
  const loading = ref(true);

  onMounted(() => {
    const subscription = liveQuery(() =>
      db.tasks.where('_deleted').notEqual(true).toArray()
    ).subscribe({
      next: (result) => {
        tasks.value = result;
        loading.value = false;
      },
      error: (err) => console.error(err),
    });

    onUnmounted(() => subscription.unsubscribe());
  });

  return { tasks, loading };
}
```

## Service Worker Background Sync

```typescript
// In service worker
import { BackgroundSyncPlugin } from 'workbox-background-sync';

const bgSyncPlugin = new BackgroundSyncPlugin('sync-queue', {
  maxRetentionTime: 24 * 60, // 24 hours
});

// Register for POST/PUT/DELETE requests
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
```

## Best Practices

1. **Generate IDs client-side** - Use UUIDs to avoid conflicts
2. **Always write locally first** - Never block on network
3. **Queue all mutations** - Even when online, for reliability
4. **Handle conflicts gracefully** - Decide on a strategy (LWW, manual, merge)
5. **Soft delete before hard delete** - Ensure delete syncs before removing
6. **Version records** - For conflict detection
7. **Batch sync operations** - Reduce API calls when possible
8. **Show sync status to users** - They need to know if data is pending
9. **Test offline thoroughly** - Use Chrome DevTools offline mode
10. **Clean up old data** - Implement TTL for cached data
