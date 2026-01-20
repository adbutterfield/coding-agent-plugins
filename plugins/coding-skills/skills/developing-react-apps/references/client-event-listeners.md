---
title: Deduplicate Global Event Listeners
impact: LOW
---

## Deduplicate Global Event Listeners

> **Review:** `addEventListener` inside `useEffect` in hooks that may be used multiple times
> **Writing:** When creating a custom hook for keyboard shortcuts, resize, or other global events
> **Technique:** Use useSWRSubscription or module-level callback tracking to share one listener across N instances

When multiple component instances use the same event hook, each registers its own listener. Use SWR's subscription pattern to share a single listener.

### Fail

```tsx
// ❌ N instances = N listeners attached to window
function useKeyboardShortcut(key: string, callback: () => void) {
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && e.key === key) {
        callback()
      }
    }
    window.addEventListener('keydown', handler)
    return () => window.removeEventListener('keydown', handler)
  }, [key, callback])
}
```

### Pass

```tsx
// ✅ N instances = 1 listener with callback routing
import useSWRSubscription from 'swr/subscription'

// Module-level Map to track callbacks per key
const keyCallbacks = new Map<string, Set<() => void>>()

function useKeyboardShortcut(key: string, callback: () => void) {
  // Register this callback in the Map
  useEffect(() => {
    if (!keyCallbacks.has(key)) {
      keyCallbacks.set(key, new Set())
    }
    keyCallbacks.get(key)!.add(callback)

    return () => {
      const set = keyCallbacks.get(key)
      if (set) {
        set.delete(callback)
        if (set.size === 0) {
          keyCallbacks.delete(key)
        }
      }
    }
  }, [key, callback])

  useSWRSubscription('global-keydown', () => {
    const handler = (e: KeyboardEvent) => {
      if (e.metaKey && keyCallbacks.has(e.key)) {
        keyCallbacks.get(e.key)!.forEach(cb => cb())
      }
    }
    window.addEventListener('keydown', handler)
    return () => window.removeEventListener('keydown', handler)
  })
}

function Profile() {
  // Multiple shortcuts share the same listener
  useKeyboardShortcut('p', () => { /* ... */ })
  useKeyboardShortcut('k', () => { /* ... */ })
  // ...
}
```

### Why This Matters

- Each event listener has memory overhead and runs on every event
- With many component instances, duplicate listeners multiply CPU work
- SWR subscription pattern ensures exactly one listener regardless of usage count

### When to Apply

- Custom hooks for global events (keyboard, resize, scroll, online/offline)
- Any hook that attaches to `window` or `document` and may be used multiple times
- Event-based state synchronization across components

### When to Skip

- Hooks used only once in the entire app
- Component-specific listeners (e.g., on a specific element ref)
- When callback execution order matters and you need separate listeners

### Expected Impact

A single listener serves N components instead of N listeners, reducing memory usage and improving performance. This is especially noticeable when many component instances share the same global event, as CPU work during event dispatch scales with listener count.

### See Also

- `client-swr-dedup.md` - Similar deduplication for data fetching
- `client-passive-event-listeners.md` - Performance optimization for scroll/touch listeners
