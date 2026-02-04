# Touch-Optimized Base Styles

## Overview

Mobile-first PWAs need touch-optimized styles for a native-like feel. This includes proper touch targets, gesture support, and preventing common mobile web pitfalls.

## Touch Target Guidelines

| Platform | Minimum Size | Recommended |
|----------|--------------|-------------|
| Apple HIG | 44×44 pt | 44×44 pt |
| Material Design | 48×48 dp | 48×48 dp |
| WCAG 2.5.5 | 44×44 CSS px | 44×44 CSS px |

**Rule**: Always aim for 44×44 CSS pixels minimum.

## Complete Touch Styles Template

```css
/* ===========================================
   BASE RESET & TOUCH OPTIMIZATION
   =========================================== */

*,
*::before,
*::after {
  box-sizing: border-box;

  /* Disable tap highlight on iOS */
  -webkit-tap-highlight-color: transparent;

  /* Prevent text size adjustment on orientation change */
  -webkit-text-size-adjust: 100%;
}

html {
  /* Prevent pull-to-refresh in standalone mode */
  overscroll-behavior-y: none;

  /* Smooth scrolling */
  scroll-behavior: smooth;

  /* Font smoothing */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  margin: 0;
  padding: 0;

  /* Prevent body scroll when modal is open */
  &.modal-open {
    overflow: hidden;
    position: fixed;
    width: 100%;
  }

  /* Prevent selection on the body (optional, be careful) */
  /* -webkit-user-select: none; */
  /* user-select: none; */
}

/* ===========================================
   TOUCH TARGETS
   =========================================== */

/* All interactive elements should be at least 44x44 */
button,
a,
input,
select,
textarea,
[role="button"],
[role="link"],
[tabindex]:not([tabindex="-1"]) {
  /* Minimum touch target */
  min-height: 44px;
  min-width: 44px;

  /* For inline elements, ensure clickable area */
  &:not(input):not(textarea):not(select) {
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }
}

/* Links that should remain inline */
p a,
span a,
li a {
  /* Extend clickable area without changing layout */
  position: relative;

  &::before {
    content: '';
    position: absolute;
    top: -8px;
    right: -8px;
    bottom: -8px;
    left: -8px;
  }
}

/* Icon buttons need explicit sizing */
.icon-btn {
  width: 44px;
  height: 44px;
  padding: 10px;

  display: inline-flex;
  align-items: center;
  justify-content: center;

  /* Visual feedback */
  border-radius: 50%;
  transition: background-color 0.15s ease;

  &:active {
    background-color: rgba(0, 0, 0, 0.1);
  }
}

/* List items as touch targets */
.list-item {
  min-height: 48px;
  padding: 12px 16px;

  display: flex;
  align-items: center;
  gap: 16px;

  /* Ripple-like feedback */
  transition: background-color 0.15s ease;

  &:active {
    background-color: rgba(0, 0, 0, 0.05);
  }
}

/* ===========================================
   TEXT SELECTION & INTERACTION
   =========================================== */

/* Prevent text selection on interactive elements */
button,
[role="button"],
.no-select {
  -webkit-user-select: none;
  user-select: none;
}

/* Allow selection on content */
p,
article,
.selectable {
  -webkit-user-select: text;
  user-select: text;
}

/* Prevent callout (long-press popup) on images */
img {
  -webkit-touch-callout: none;
}

/* Allow callout on links */
a {
  -webkit-touch-callout: default;
}

/* ===========================================
   SCROLLING
   =========================================== */

/* Smooth native scrolling */
.scrollable {
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;

  /* Prevent scroll chaining */
  overscroll-behavior: contain;
}

/* Horizontal scroll */
.scroll-horizontal {
  overflow-x: auto;
  overflow-y: hidden;
  -webkit-overflow-scrolling: touch;

  /* Hide scrollbar but keep functionality */
  scrollbar-width: none;
  -ms-overflow-style: none;

  &::-webkit-scrollbar {
    display: none;
  }

  /* Scroll snap for carousels */
  scroll-snap-type: x mandatory;

  & > * {
    scroll-snap-align: start;
  }
}

/* Pull-to-refresh container */
.pull-to-refresh-container {
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  overscroll-behavior-y: contain;
}

/* ===========================================
   INPUTS & FORMS
   =========================================== */

/* Larger input fields for touch */
input,
select,
textarea {
  /* iOS zoom prevention */
  font-size: 16px;

  /* Touch-friendly size */
  min-height: 48px;
  padding: 12px 16px;

  /* Full width by default */
  width: 100%;

  /* Better borders */
  border: 1px solid #d1d5db;
  border-radius: 8px;

  /* Prevent zoom on focus (iOS) */
  &:focus {
    font-size: 16px;
  }
}

/* Textarea specifics */
textarea {
  min-height: 120px;
  resize: vertical;
}

/* Select dropdown */
select {
  /* Remove default arrow styling */
  appearance: none;
  background-image: url("data:image/svg+xml,..."); /* Add custom arrow */
  background-repeat: no-repeat;
  background-position: right 12px center;
  padding-right: 40px;
}

/* Checkbox and radio */
input[type="checkbox"],
input[type="radio"] {
  width: 24px;
  height: 24px;
  min-width: 24px;
  min-height: 24px;

  /* Touch-friendly margin */
  margin: 10px;
}

/* Search input */
input[type="search"] {
  /* Remove Safari styling */
  -webkit-appearance: none;

  &::-webkit-search-cancel-button {
    -webkit-appearance: none;
    height: 24px;
    width: 24px;
    cursor: pointer;
  }
}

/* ===========================================
   BUTTONS
   =========================================== */

.btn {
  /* Reset */
  appearance: none;
  border: none;
  background: none;
  cursor: pointer;

  /* Touch target */
  min-height: 44px;
  min-width: 64px;
  padding: 12px 24px;

  /* Typography */
  font-size: 16px;
  font-weight: 500;
  text-align: center;
  white-space: nowrap;

  /* Layout */
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;

  /* Visual */
  border-radius: 8px;
  transition: background-color 0.15s ease, transform 0.1s ease;

  /* Touch feedback */
  &:active {
    transform: scale(0.97);
  }

  /* Disabled state */
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    transform: none;
  }
}

/* Primary button */
.btn-primary {
  background-color: var(--primary-color, #3b82f6);
  color: white;

  &:active:not(:disabled) {
    background-color: var(--primary-dark, #2563eb);
  }
}

/* Secondary button */
.btn-secondary {
  background-color: transparent;
  color: var(--primary-color, #3b82f6);
  border: 1px solid var(--primary-color, #3b82f6);

  &:active:not(:disabled) {
    background-color: rgba(59, 130, 246, 0.1);
  }
}

/* Full-width button */
.btn-block {
  width: 100%;
}

/* ===========================================
   TOUCH FEEDBACK
   =========================================== */

/* Ripple effect base */
.ripple {
  position: relative;
  overflow: hidden;

  &::after {
    content: '';
    position: absolute;
    inset: 0;
    background: radial-gradient(
      circle,
      rgba(0, 0, 0, 0.1) 10%,
      transparent 10%
    );
    background-size: 100%;
    opacity: 0;
    transition: background-size 0.3s ease, opacity 0.3s ease;
  }

  &:active::after {
    background-size: 10000%;
    opacity: 1;
    transition: 0s;
  }
}

/* Simple press feedback */
.pressable {
  transition: opacity 0.1s ease, transform 0.1s ease;

  &:active {
    opacity: 0.7;
    transform: scale(0.98);
  }
}

/* Highlight on touch */
.highlight-on-touch {
  transition: background-color 0.15s ease;

  &:active {
    background-color: rgba(0, 0, 0, 0.05);
  }
}

/* ===========================================
   GESTURES
   =========================================== */

/* Swipeable item */
.swipeable {
  touch-action: pan-y; /* Allow vertical scroll, capture horizontal */
  overflow-x: hidden;
}

/* Drag handle */
.drag-handle {
  cursor: grab;
  touch-action: none; /* Prevent scrolling when dragging */

  &:active {
    cursor: grabbing;
  }
}

/* Pinch-zoom container */
.pinch-zoom {
  touch-action: none; /* Handle all touch events in JS */
  overflow: hidden;
}

/* Disable pinch zoom globally (if needed) */
.no-pinch-zoom {
  touch-action: pan-x pan-y;
}

/* ===========================================
   CARDS & SURFACES
   =========================================== */

.card {
  background: white;
  border-radius: 12px;
  padding: 16px;

  /* Subtle shadow */
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);

  /* Interactive card */
  &.card-interactive {
    cursor: pointer;
    transition: transform 0.15s ease, box-shadow 0.15s ease;

    &:active {
      transform: scale(0.98);
      box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
    }
  }
}

/* ===========================================
   SPACING FOR TOUCH
   =========================================== */

/* Ensure adequate spacing between touch targets */
.touch-list {
  & > * + * {
    margin-top: 8px; /* Minimum 8px gap between touch targets */
  }
}

/* Horizontal button group */
.btn-group {
  display: flex;
  gap: 8px; /* Minimum spacing */

  & > .btn {
    flex: 1;
  }
}

/* ===========================================
   LOADING & DISABLED STATES
   =========================================== */

.loading {
  pointer-events: none;
  opacity: 0.7;
}

/* Loading spinner inside button */
.btn.loading::before {
  content: '';
  width: 16px;
  height: 16px;
  border: 2px solid currentColor;
  border-right-color: transparent;
  border-radius: 50%;
  animation: spin 0.75s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* ===========================================
   DARK MODE ADJUSTMENTS
   =========================================== */

@media (prefers-color-scheme: dark) {
  .icon-btn:active {
    background-color: rgba(255, 255, 255, 0.1);
  }

  .list-item:active {
    background-color: rgba(255, 255, 255, 0.05);
  }

  .highlight-on-touch:active {
    background-color: rgba(255, 255, 255, 0.05);
  }

  .card {
    background: #1f2937;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
  }
}

/* ===========================================
   ANIMATIONS
   =========================================== */

/* Reduce motion for accessibility */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Page transitions */
.page-enter {
  opacity: 0;
  transform: translateX(20px);
}

.page-enter-active {
  opacity: 1;
  transform: translateX(0);
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.page-exit {
  opacity: 1;
  transform: translateX(0);
}

.page-exit-active {
  opacity: 0;
  transform: translateX(-20px);
  transition: opacity 0.2s ease, transform 0.2s ease;
}
```

## JavaScript Touch Utilities

```typescript
// touch-utils.ts

/**
 * Detect if device supports touch
 */
export function isTouchDevice(): boolean {
  return (
    'ontouchstart' in window ||
    navigator.maxTouchPoints > 0 ||
    // @ts-ignore
    navigator.msMaxTouchPoints > 0
  );
}

/**
 * Get touch/mouse position from event
 */
export function getEventPosition(
  event: TouchEvent | MouseEvent
): { x: number; y: number } {
  if ('touches' in event) {
    return {
      x: event.touches[0].clientX,
      y: event.touches[0].clientY,
    };
  }
  return {
    x: event.clientX,
    y: event.clientY,
  };
}

/**
 * Simple swipe detection
 */
export function detectSwipe(
  element: HTMLElement,
  callback: (direction: 'left' | 'right' | 'up' | 'down') => void,
  threshold = 50
): () => void {
  let startX = 0;
  let startY = 0;

  const handleStart = (e: TouchEvent) => {
    startX = e.touches[0].clientX;
    startY = e.touches[0].clientY;
  };

  const handleEnd = (e: TouchEvent) => {
    const endX = e.changedTouches[0].clientX;
    const endY = e.changedTouches[0].clientY;

    const diffX = endX - startX;
    const diffY = endY - startY;

    if (Math.abs(diffX) > Math.abs(diffY) && Math.abs(diffX) > threshold) {
      callback(diffX > 0 ? 'right' : 'left');
    } else if (Math.abs(diffY) > threshold) {
      callback(diffY > 0 ? 'down' : 'up');
    }
  };

  element.addEventListener('touchstart', handleStart, { passive: true });
  element.addEventListener('touchend', handleEnd, { passive: true });

  return () => {
    element.removeEventListener('touchstart', handleStart);
    element.removeEventListener('touchend', handleEnd);
  };
}

/**
 * Long press detection
 */
export function detectLongPress(
  element: HTMLElement,
  callback: () => void,
  duration = 500
): () => void {
  let timeout: ReturnType<typeof setTimeout>;
  let isLongPress = false;

  const start = () => {
    isLongPress = false;
    timeout = setTimeout(() => {
      isLongPress = true;
      callback();
    }, duration);
  };

  const cancel = () => {
    clearTimeout(timeout);
  };

  const preventClick = (e: Event) => {
    if (isLongPress) {
      e.preventDefault();
      e.stopPropagation();
    }
  };

  element.addEventListener('touchstart', start, { passive: true });
  element.addEventListener('touchend', cancel, { passive: true });
  element.addEventListener('touchmove', cancel, { passive: true });
  element.addEventListener('click', preventClick, { capture: true });

  return () => {
    clearTimeout(timeout);
    element.removeEventListener('touchstart', start);
    element.removeEventListener('touchend', cancel);
    element.removeEventListener('touchmove', cancel);
    element.removeEventListener('click', preventClick);
  };
}

/**
 * Add haptic feedback (if supported)
 */
export function hapticFeedback(
  type: 'light' | 'medium' | 'heavy' = 'medium'
): void {
  if ('vibrate' in navigator) {
    const patterns = {
      light: [10],
      medium: [20],
      heavy: [30],
    };
    navigator.vibrate(patterns[type]);
  }
}

/**
 * Prevent bounce scroll on iOS
 */
export function preventBounceScroll(element: HTMLElement): () => void {
  let startY = 0;

  const handleTouchStart = (e: TouchEvent) => {
    startY = e.touches[0].clientY;
  };

  const handleTouchMove = (e: TouchEvent) => {
    const currentY = e.touches[0].clientY;
    const scrollTop = element.scrollTop;
    const scrollHeight = element.scrollHeight;
    const clientHeight = element.clientHeight;

    const isAtTop = scrollTop === 0 && currentY > startY;
    const isAtBottom =
      scrollTop + clientHeight >= scrollHeight && currentY < startY;

    if (isAtTop || isAtBottom) {
      e.preventDefault();
    }
  };

  element.addEventListener('touchstart', handleTouchStart, { passive: true });
  element.addEventListener('touchmove', handleTouchMove, { passive: false });

  return () => {
    element.removeEventListener('touchstart', handleTouchStart);
    element.removeEventListener('touchmove', handleTouchMove);
  };
}
```

## React Touch Components

```tsx
// Pressable.tsx
import React, { useState, ReactNode } from 'react';

interface PressableProps {
  children: ReactNode;
  onPress?: () => void;
  onLongPress?: () => void;
  disabled?: boolean;
  className?: string;
}

export function Pressable({
  children,
  onPress,
  onLongPress,
  disabled,
  className = '',
}: PressableProps) {
  const [isPressed, setIsPressed] = useState(false);

  return (
    <div
      role="button"
      tabIndex={disabled ? -1 : 0}
      className={`pressable ${className} ${isPressed ? 'pressed' : ''} ${
        disabled ? 'disabled' : ''
      }`}
      onTouchStart={() => setIsPressed(true)}
      onTouchEnd={() => setIsPressed(false)}
      onMouseDown={() => setIsPressed(true)}
      onMouseUp={() => setIsPressed(false)}
      onMouseLeave={() => setIsPressed(false)}
      onClick={disabled ? undefined : onPress}
      aria-disabled={disabled}
    >
      {children}
    </div>
  );
}
```

## Best Practices

1. **Always test on real devices** - Simulators miss touch nuances
2. **Use `passive: true`** for scroll listeners when possible
3. **Provide visual feedback** immediately on touch
4. **Don't rely on hover states** - they don't exist on touch
5. **Account for fat fingers** - make targets larger than minimum
6. **Prevent double-tap zoom** - use `touch-action: manipulation`
7. **Test with assistive technologies** - ensure accessibility
