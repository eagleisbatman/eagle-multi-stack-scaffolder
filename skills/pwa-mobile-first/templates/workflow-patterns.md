# Workflow App Patterns

## Overview

Workflow apps (task managers, inspection forms, field data collection) require specific UX patterns for a native-like experience. These patterns are essential for PWAs that need to work offline and sync data.

## 1. Optimistic UI Updates

Update the UI immediately before the server confirms the change. Revert if the request fails.

### React Implementation

```tsx
// hooks/useOptimisticMutation.ts
import { useState, useCallback } from 'react';

interface UseOptimisticMutationOptions<T, U> {
  mutationFn: (data: U) => Promise<T>;
  onOptimisticUpdate: (data: U) => void;
  onError: (error: Error, data: U) => void;
  onSuccess?: (result: T, data: U) => void;
}

export function useOptimisticMutation<T, U>({
  mutationFn,
  onOptimisticUpdate,
  onError,
  onSuccess,
}: UseOptimisticMutationOptions<T, U>) {
  const [isLoading, setIsLoading] = useState(false);

  const mutate = useCallback(
    async (data: U) => {
      // Apply optimistic update immediately
      onOptimisticUpdate(data);

      setIsLoading(true);
      try {
        const result = await mutationFn(data);
        onSuccess?.(result, data);
        return result;
      } catch (error) {
        // Revert on error
        onError(error as Error, data);
        throw error;
      } finally {
        setIsLoading(false);
      }
    },
    [mutationFn, onOptimisticUpdate, onError, onSuccess]
  );

  return { mutate, isLoading };
}
```

### Complete Task Toggle Example

```tsx
// components/TaskItem.tsx
import { useState } from 'react';
import { useOptimisticMutation } from '@/hooks/useOptimisticMutation';
import { updateTask } from '@/lib/api';
import { Task } from '@/lib/types';

interface TaskItemProps {
  task: Task;
  onUpdate: (task: Task) => void;
}

export function TaskItem({ task, onUpdate }: TaskItemProps) {
  const [localTask, setLocalTask] = useState(task);
  const [error, setError] = useState<string | null>(null);

  const { mutate, isLoading } = useOptimisticMutation({
    mutationFn: (updatedTask: Task) => updateTask(updatedTask.id, updatedTask),

    onOptimisticUpdate: (updatedTask) => {
      setLocalTask(updatedTask);
      setError(null);
    },

    onError: (err, updatedTask) => {
      // Revert to original
      setLocalTask(task);
      setError('Failed to save. Tap to retry.');
    },

    onSuccess: (result) => {
      setLocalTask(result);
      onUpdate(result);
    },
  });

  const toggleComplete = () => {
    mutate({
      ...localTask,
      status: localTask.status === 'done' ? 'todo' : 'done',
      updatedAt: Date.now(),
    });
  };

  return (
    <div
      className={`task-item ${localTask.status === 'done' ? 'completed' : ''}`}
      onClick={error ? toggleComplete : undefined}
    >
      <button
        onClick={toggleComplete}
        disabled={isLoading}
        className="task-checkbox"
        aria-label={localTask.status === 'done' ? 'Mark incomplete' : 'Mark complete'}
      >
        {isLoading ? (
          <span className="spinner" />
        ) : (
          <span className={localTask.status === 'done' ? 'checked' : 'unchecked'} />
        )}
      </button>

      <span className="task-title">{localTask.title}</span>

      {error && (
        <span className="error-badge" title={error}>
          ⚠️
        </span>
      )}
    </div>
  );
}
```

### With React Query

```tsx
// hooks/useUpdateTask.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { updateTask } from '@/lib/api';
import { Task } from '@/lib/types';

export function useUpdateTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<Task> }) =>
      updateTask(id, data),

    // Optimistic update
    onMutate: async ({ id, data }) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['tasks'] });

      // Snapshot current state
      const previousTasks = queryClient.getQueryData<Task[]>(['tasks']);

      // Optimistically update
      queryClient.setQueryData<Task[]>(['tasks'], (old) =>
        old?.map((task) =>
          task.id === id ? { ...task, ...data, updatedAt: Date.now() } : task
        )
      );

      // Return snapshot for rollback
      return { previousTasks };
    },

    // Rollback on error
    onError: (err, variables, context) => {
      if (context?.previousTasks) {
        queryClient.setQueryData(['tasks'], context.previousTasks);
      }
    },

    // Refetch after success/error
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });
}
```

---

## 2. Offline Form Queue

Queue form submissions when offline and automatically submit when back online.

### Implementation

```typescript
// lib/offline/form-queue.ts
import { db, SyncQueueItem } from './db';

export interface QueuedForm {
  id?: number;
  formType: string;
  data: Record<string, unknown>;
  createdAt: number;
  retryCount: number;
  status: 'pending' | 'submitting' | 'failed';
  lastError?: string;
}

class FormQueue {
  private isProcessing = false;
  private listeners = new Set<(forms: QueuedForm[]) => void>();

  /**
   * Add form to queue
   */
  async add(formType: string, data: Record<string, unknown>): Promise<number> {
    const id = await db.formQueue.add({
      formType,
      data,
      createdAt: Date.now(),
      retryCount: 0,
      status: 'pending',
    });

    this.notifyListeners();

    // Try to submit if online
    if (navigator.onLine) {
      this.process();
    }

    return id;
  }

  /**
   * Process all pending forms
   */
  async process(): Promise<void> {
    if (this.isProcessing || !navigator.onLine) return;

    this.isProcessing = true;

    try {
      const forms = await db.formQueue
        .where('status')
        .anyOf(['pending', 'failed'])
        .toArray();

      for (const form of forms) {
        await this.submitForm(form);
      }
    } finally {
      this.isProcessing = false;
      this.notifyListeners();
    }
  }

  private async submitForm(form: QueuedForm): Promise<void> {
    try {
      // Mark as submitting
      await db.formQueue.update(form.id!, { status: 'submitting' });
      this.notifyListeners();

      // Submit to API
      const response = await fetch(`/api/${form.formType}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(form.data),
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      // Success - remove from queue
      await db.formQueue.delete(form.id!);

    } catch (error) {
      // Update retry count and status
      const newRetryCount = form.retryCount + 1;

      if (newRetryCount >= 5) {
        // Max retries - mark as permanently failed
        await db.formQueue.update(form.id!, {
          status: 'failed',
          retryCount: newRetryCount,
          lastError: String(error),
        });
      } else {
        // Will retry
        await db.formQueue.update(form.id!, {
          status: 'pending',
          retryCount: newRetryCount,
          lastError: String(error),
        });
      }
    }
  }

  /**
   * Get all queued forms
   */
  async getAll(): Promise<QueuedForm[]> {
    return db.formQueue.toArray();
  }

  /**
   * Get pending count
   */
  async getPendingCount(): Promise<number> {
    return db.formQueue.where('status').equals('pending').count();
  }

  /**
   * Retry a failed form
   */
  async retry(id: number): Promise<void> {
    await db.formQueue.update(id, { status: 'pending', retryCount: 0 });
    this.process();
  }

  /**
   * Remove a form from queue
   */
  async remove(id: number): Promise<void> {
    await db.formQueue.delete(id);
    this.notifyListeners();
  }

  /**
   * Subscribe to queue changes
   */
  subscribe(listener: (forms: QueuedForm[]) => void): () => void {
    this.listeners.add(listener);
    this.getAll().then(listener);
    return () => this.listeners.delete(listener);
  }

  private async notifyListeners(): Promise<void> {
    const forms = await this.getAll();
    this.listeners.forEach((l) => l(forms));
  }
}

export const formQueue = new FormQueue();

// Auto-process when coming online
if (typeof window !== 'undefined') {
  window.addEventListener('online', () => formQueue.process());
}
```

### React Form Component

```tsx
// components/OfflineForm.tsx
import { useState } from 'react';
import { formQueue } from '@/lib/offline/form-queue';

interface OfflineFormProps {
  formType: string;
  children: React.ReactNode;
  onSubmit: (data: FormData) => Record<string, unknown>;
  onSuccess?: () => void;
}

export function OfflineForm({
  formType,
  children,
  onSubmit,
  onSuccess,
}: OfflineFormProps) {
  const [status, setStatus] = useState<'idle' | 'queued' | 'error'>('idle');
  const [isOnline] = useState(navigator.onLine);

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();

    const formData = new FormData(e.currentTarget);
    const data = onSubmit(formData);

    try {
      await formQueue.add(formType, data);
      setStatus('queued');

      // Reset form
      e.currentTarget.reset();

      // Show success message
      onSuccess?.();

      // Reset status after delay
      setTimeout(() => setStatus('idle'), 3000);
    } catch (error) {
      setStatus('error');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {children}

      {!isOnline && (
        <div className="offline-notice">
          You're offline. Form will be submitted when you're back online.
        </div>
      )}

      {status === 'queued' && (
        <div className="success-notice">
          ✓ Saved{!navigator.onLine && ' (will sync when online)'}
        </div>
      )}

      {status === 'error' && (
        <div className="error-notice">Failed to save. Please try again.</div>
      )}
    </form>
  );
}
```

### Pending Forms Indicator

```tsx
// components/PendingFormsIndicator.tsx
import { useState, useEffect } from 'react';
import { formQueue, QueuedForm } from '@/lib/offline/form-queue';

export function PendingFormsIndicator() {
  const [forms, setForms] = useState<QueuedForm[]>([]);

  useEffect(() => {
    return formQueue.subscribe(setForms);
  }, []);

  const pendingCount = forms.filter((f) => f.status === 'pending').length;
  const failedCount = forms.filter((f) => f.status === 'failed').length;

  if (forms.length === 0) return null;

  return (
    <div className="pending-forms-indicator">
      {pendingCount > 0 && (
        <span className="pending">
          {pendingCount} form{pendingCount !== 1 ? 's' : ''} pending sync
        </span>
      )}
      {failedCount > 0 && (
        <span className="failed">
          {failedCount} failed -{' '}
          <button onClick={() => forms.forEach((f) => formQueue.retry(f.id!))}>
            Retry all
          </button>
        </span>
      )}
    </div>
  );
}
```

---

## 3. Pull-to-Refresh

Native-feeling pull gesture to refresh content.

### React Implementation

```tsx
// hooks/usePullToRefresh.ts
import { useRef, useState, useEffect, useCallback } from 'react';

interface UsePullToRefreshOptions {
  onRefresh: () => Promise<void>;
  threshold?: number;
  resistance?: number;
}

export function usePullToRefresh({
  onRefresh,
  threshold = 80,
  resistance = 2.5,
}: UsePullToRefreshOptions) {
  const containerRef = useRef<HTMLDivElement>(null);
  const [isRefreshing, setIsRefreshing] = useState(false);
  const [pullDistance, setPullDistance] = useState(0);

  const startY = useRef(0);
  const currentY = useRef(0);
  const isPulling = useRef(false);

  const handleTouchStart = useCallback((e: TouchEvent) => {
    const container = containerRef.current;
    if (!container || container.scrollTop > 0) return;

    startY.current = e.touches[0].clientY;
    isPulling.current = true;
  }, []);

  const handleTouchMove = useCallback(
    (e: TouchEvent) => {
      if (!isPulling.current) return;

      const container = containerRef.current;
      if (!container || container.scrollTop > 0) {
        isPulling.current = false;
        setPullDistance(0);
        return;
      }

      currentY.current = e.touches[0].clientY;
      const distance = currentY.current - startY.current;

      if (distance > 0) {
        e.preventDefault();
        // Apply resistance
        const adjustedDistance = Math.min(distance / resistance, threshold * 1.5);
        setPullDistance(adjustedDistance);
      }
    },
    [threshold, resistance]
  );

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

### Pull-to-Refresh Component

```tsx
// components/PullToRefresh.tsx
import { ReactNode } from 'react';
import { usePullToRefresh } from '@/hooks/usePullToRefresh';

interface PullToRefreshProps {
  onRefresh: () => Promise<void>;
  children: ReactNode;
}

export function PullToRefresh({ onRefresh, children }: PullToRefreshProps) {
  const { containerRef, isRefreshing, pullDistance, pullProgress } =
    usePullToRefresh({ onRefresh });

  return (
    <div ref={containerRef} className="pull-to-refresh-container">
      {/* Pull indicator */}
      <div
        className="pull-indicator"
        style={{
          height: `${pullDistance}px`,
          opacity: pullProgress,
        }}
      >
        <div
          className="pull-spinner"
          style={{
            transform: `rotate(${pullProgress * 360}deg)`,
          }}
        >
          {isRefreshing ? (
            <span className="spinner-active" />
          ) : (
            <span className="arrow">↓</span>
          )}
        </div>
        <span className="pull-text">
          {isRefreshing
            ? 'Refreshing...'
            : pullProgress >= 1
            ? 'Release to refresh'
            : 'Pull to refresh'}
        </span>
      </div>

      {/* Content */}
      <div className="pull-content">{children}</div>
    </div>
  );
}
```

### CSS

```css
.pull-to-refresh-container {
  height: 100%;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  overscroll-behavior-y: contain;
}

.pull-indicator {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  background: #f3f4f6;
}

.pull-spinner {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.1s ease;
}

.spinner-active {
  width: 24px;
  height: 24px;
  border: 2px solid #3b82f6;
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 0.75s linear infinite;
}

.arrow {
  font-size: 20px;
  color: #6b7280;
}

.pull-text {
  font-size: 12px;
  color: #6b7280;
  margin-top: 4px;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
```

---

## 4. Skeleton Loaders

Placeholder UI during data loading for perceived performance.

### Skeleton Components

```tsx
// components/ui/Skeleton.tsx
import { ReactNode } from 'react';

interface SkeletonProps {
  width?: string | number;
  height?: string | number;
  borderRadius?: string | number;
  className?: string;
}

export function Skeleton({
  width = '100%',
  height = '1em',
  borderRadius = 4,
  className = '',
}: SkeletonProps) {
  return (
    <div
      className={`skeleton ${className}`}
      style={{
        width,
        height,
        borderRadius,
      }}
    />
  );
}

// Skeleton text line
export function SkeletonText({
  lines = 1,
  lastLineWidth = '60%',
}: {
  lines?: number;
  lastLineWidth?: string;
}) {
  return (
    <div className="skeleton-text">
      {Array.from({ length: lines }).map((_, i) => (
        <Skeleton
          key={i}
          height="1em"
          width={i === lines - 1 ? lastLineWidth : '100%'}
          className="skeleton-line"
        />
      ))}
    </div>
  );
}

// Skeleton avatar
export function SkeletonAvatar({ size = 40 }: { size?: number }) {
  return <Skeleton width={size} height={size} borderRadius="50%" />;
}

// Skeleton card
export function SkeletonCard() {
  return (
    <div className="skeleton-card">
      <div className="skeleton-card-header">
        <SkeletonAvatar />
        <div className="skeleton-card-meta">
          <Skeleton width="60%" height="1em" />
          <Skeleton width="40%" height="0.875em" />
        </div>
      </div>
      <SkeletonText lines={3} />
    </div>
  );
}

// Skeleton list item
export function SkeletonListItem() {
  return (
    <div className="skeleton-list-item">
      <Skeleton width={24} height={24} borderRadius={4} />
      <div className="skeleton-list-content">
        <Skeleton width="70%" height="1em" />
        <Skeleton width="40%" height="0.875em" />
      </div>
    </div>
  );
}

// Skeleton list
export function SkeletonList({ count = 5 }: { count?: number }) {
  return (
    <div className="skeleton-list">
      {Array.from({ length: count }).map((_, i) => (
        <SkeletonListItem key={i} />
      ))}
    </div>
  );
}
```

### CSS

```css
/* Skeleton base */
.skeleton {
  background: linear-gradient(
    90deg,
    #e5e7eb 25%,
    #f3f4f6 50%,
    #e5e7eb 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}

/* Skeleton text */
.skeleton-text {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.skeleton-line {
  margin-bottom: 0;
}

/* Skeleton card */
.skeleton-card {
  padding: 16px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.skeleton-card-header {
  display: flex;
  gap: 12px;
  margin-bottom: 16px;
}

.skeleton-card-meta {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
  justify-content: center;
}

/* Skeleton list item */
.skeleton-list-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 0;
  border-bottom: 1px solid #e5e7eb;
}

.skeleton-list-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  .skeleton {
    background: linear-gradient(
      90deg,
      #374151 25%,
      #4b5563 50%,
      #374151 75%
    );
    background-size: 200% 100%;
  }

  .skeleton-card {
    background: #1f2937;
  }

  .skeleton-list-item {
    border-bottom-color: #374151;
  }
}
```

### Usage Example

```tsx
// pages/TaskList.tsx
import { useState, useEffect } from 'react';
import { SkeletonList } from '@/components/ui/Skeleton';
import { TaskItem } from '@/components/TaskItem';
import { PullToRefresh } from '@/components/PullToRefresh';
import { useTasks } from '@/hooks/useTasks';

export function TaskList() {
  const { tasks, loading, refetch } = useTasks();

  return (
    <PullToRefresh onRefresh={refetch}>
      {loading ? (
        <SkeletonList count={5} />
      ) : tasks.length === 0 ? (
        <EmptyState message="No tasks yet" />
      ) : (
        <ul className="task-list">
          {tasks.map((task) => (
            <TaskItem key={task.id} task={task} />
          ))}
        </ul>
      )}
    </PullToRefresh>
  );
}
```

---

## Best Practices Summary

1. **Optimistic Updates**: Always update UI immediately, revert on error
2. **Show Sync Status**: Users need to know if data is pending
3. **Queue Everything**: Even when online, queue ensures reliability
4. **Provide Feedback**: Visual indicators for all async operations
5. **Handle Errors Gracefully**: Show error states with retry options
6. **Test Offline Thoroughly**: Simulate slow/offline in DevTools
7. **Use Skeleton Loaders**: Better perceived performance than spinners
8. **Respect User's Time**: Don't block UI for network operations
