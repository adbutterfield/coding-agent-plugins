---
title: Store Event Handlers in Refs
impact: LOW
---

## Store Event Handlers in Refs

> **Review:** `useEffect` with callback in dependency array that adds/removes event listeners
> **Writing:** Creating a custom hook that subscribes to events with a user-provided callback
> **Technique:** Store the callback in a ref and update it in a separate effect, keeping the subscription effect stable

Store callbacks in refs when used in effects that shouldn't re-subscribe on callback changes. This prevents unnecessary listener teardown/setup cycles that can cause performance issues and missed events.

### Fail

```tsx
// ❌ Re-subscribes on every render when handler identity changes
function useWindowEvent(event: string, handler: (e) => void) {
  useEffect(() => {
    window.addEventListener(event, handler)
    return () => window.removeEventListener(event, handler)
  }, [event, handler])
}
```

### Pass

```tsx
// ✅ Stable subscription - only re-subscribes when event type changes
function useWindowEvent(event: string, handler: (e) => void) {
  const handlerRef = useRef(handler)
  useEffect(() => {
    handlerRef.current = handler
  }, [handler])

  useEffect(() => {
    const listener = (e) => handlerRef.current(e)
    window.addEventListener(event, listener)
    return () => window.removeEventListener(event, listener)
  }, [event])
}
```

### Alternative: useEffectEvent

```tsx
// ✅ Cleaner API available in React 19+
import { useEffectEvent } from 'react'

function useWindowEvent(event: string, handler: (e) => void) {
  const onEvent = useEffectEvent(handler)

  useEffect(() => {
    window.addEventListener(event, onEvent)
    return () => window.removeEventListener(event, onEvent)
  }, [event])
}
```

`useEffectEvent` provides a cleaner API for the same pattern: it creates a stable function reference that always calls the latest version of the handler.

### Why This Matters

- Event listener setup/teardown is expensive and can cause missed events during the gap
- Callbacks passed as props typically have unstable identity (new function each render)
- The ref pattern decouples "what to do" from "when to subscribe"

### When to Apply

- Custom hooks that subscribe to browser events (window, document, media queries)
- WebSocket or EventSource message handlers
- Intersection/Mutation/Resize observer callbacks
- Any effect where subscription cost is high but callback logic changes frequently

### When to Skip

- Simple effects where re-running is cheap and correct behavior
- When using React 19+ with `useEffectEvent` available
- One-time effects that only run on mount (`[]` dependencies)
- When the callback identity is already stable (e.g., from `useCallback` with stable deps)

### Expected Impact

Provides stable callback references for subscriptions and event listeners, preventing unnecessary cleanup/re-subscribe cycles. This eliminates the performance overhead of tearing down and recreating listeners on every render, and prevents subtle bugs where events can be missed during the brief gap between unsubscribe and resubscribe.

### See Also

- `advanced-use-latest.md` - Generic hook for accessing latest values without triggering effects
- `rerender-dependencies.md` - Narrowing dependencies to minimize effect re-runs
