---
title: Use Passive Event Listeners for Scrolling Performance
impact: MEDIUM
---

## Use Passive Event Listeners for Scrolling Performance

> **Review:** `addEventListener('touchstart'` or `addEventListener('wheel'` without `{ passive: true }`
> **Writing:** When adding touch or wheel event listeners for tracking/analytics
> **Technique:** Add `{ passive: true }` to touch and wheel listeners that don't call preventDefault()

Add `{ passive: true }` to touch and wheel event listeners to enable immediate scrolling. Browsers normally wait for listeners to finish to check if `preventDefault()` is called, causing scroll delay.

### Fail

```typescript
// ❌ Browser waits to see if preventDefault() is called, delaying scroll
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)

  document.addEventListener('touchstart', handleTouch)
  document.addEventListener('wheel', handleWheel)

  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])
```

### Pass

```typescript
// ✅ Browser scrolls immediately, listener runs in parallel
useEffect(() => {
  const handleTouch = (e: TouchEvent) => console.log(e.touches[0].clientX)
  const handleWheel = (e: WheelEvent) => console.log(e.deltaY)

  document.addEventListener('touchstart', handleTouch, { passive: true })
  document.addEventListener('wheel', handleWheel, { passive: true })

  return () => {
    document.removeEventListener('touchstart', handleTouch)
    document.removeEventListener('wheel', handleWheel)
  }
}, [])
```

### Why This Matters

- Passive listeners tell the browser scrolling can proceed without waiting
- Eliminates scroll jank on mobile devices especially
- Chrome DevTools warns about non-passive listeners on these events

### When to Apply

- Scroll/touch tracking and analytics
- Logging user interactions
- Any listener that only reads event data without preventing default behavior

### When to Skip

- Custom swipe gestures that need to prevent scrolling
- Custom zoom/pinch controls
- Pull-to-refresh implementations
- Any listener that must call `preventDefault()`

### Expected Impact

Eliminates scroll and touch delay caused by the browser waiting to see if `preventDefault()` will be called. This removes perceptible input lag, especially on mobile devices where the browser typically waits up to 100ms before scrolling.

### See Also

- `client-event-listeners.md` - Deduplicating global event listeners
