---
title: Use Transitions for Non-Urgent Updates
impact: MEDIUM
---

## Use Transitions for Non-Urgent Updates

> **Review:** `setState` inside scroll, resize, or mousemove event handlers without `startTransition`
> **Writing:** Tracking continuous values (scroll position, mouse coordinates) that update UI non-critically
> **Technique:** Wrap frequent state updates in `startTransition(() => setState(...))`

Mark frequent, non-urgent state updates as transitions to maintain UI responsiveness. Transitions tell React that the update can be interrupted, allowing urgent updates (like typing) to take priority.

### Fail

```tsx
// ❌ Blocks UI on every scroll event, can cause jank
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0)
  useEffect(() => {
    const handler = () => setScrollY(window.scrollY)
    window.addEventListener('scroll', handler, { passive: true })
    return () => window.removeEventListener('scroll', handler)
  }, [])
}
```

### Pass

```tsx
// ✅ Non-blocking updates, UI stays responsive during rapid scrolling
import { startTransition } from 'react'

function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0)
  useEffect(() => {
    const handler = () => {
      startTransition(() => setScrollY(window.scrollY))
    }
    window.addEventListener('scroll', handler, { passive: true })
    return () => window.removeEventListener('scroll', handler)
  }, [])
}
```

### Why This Matters

- Scroll, resize, and mouse events can fire 60+ times per second
- Without transitions, each update blocks the main thread
- Transitions allow React to batch and defer these updates, keeping the UI responsive

### When to Apply

- Scroll position tracking for parallax effects or scroll-based animations
- Mouse/touch position tracking for cursors or drag indicators
- Window resize handlers that update layout state
- Any high-frequency event where the visual update can lag slightly

### When to Skip

- The update is urgent and must be immediate (e.g., form input values)
- The component is simple enough that updates don't cause noticeable lag
- You're already throttling/debouncing the event handler
- The state drives critical UI that users expect to be instant

### Expected Impact

Maintains UI responsiveness during expensive state updates by keeping urgent interactions snappy. Transitions allow React to interrupt and deprioritize non-critical updates, ensuring that user input and other high-priority interactions remain fluid even during rapid event firing.

### See Also

- `rerender-derived-state.md` - Subscribe to derived state to reduce update frequency
- `rerender-defer-reads.md` - Defer reads entirely when values aren't needed for render
