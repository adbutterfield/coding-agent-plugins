---
title: Subscribe to Derived State
impact: MEDIUM
---

## Subscribe to Derived State

> **Review:** `useWindowWidth` or similar continuous-value hooks where only a boolean/threshold matters
> **Writing:** Building responsive behavior that depends on crossing a threshold (mobile breakpoint, count > 0, etc.)
> **Technique:** Use hooks that return derived boolean state like `useMediaQuery` instead of continuous values

Subscribe to derived boolean state instead of continuous values to reduce re-render frequency. When you only care about crossing a threshold, subscribing to the raw value causes unnecessary re-renders.

### Fail

```tsx
// ❌ Re-renders on every pixel change (width=1024, 1023, 1022...)
function Sidebar() {
  const width = useWindowWidth()  // updates continuously
  const isMobile = width < 768
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```

### Pass

```tsx
// ✅ Re-renders only when crossing the 768px threshold (true <-> false)
function Sidebar() {
  const isMobile = useMediaQuery('(max-width: 767px)')
  return <nav className={isMobile ? 'mobile' : 'desktop'} />
}
```

### Why This Matters

- Continuous values can change hundreds of times per second (resize, scroll, mouse move)
- Boolean derived state collapses many changes into rare transitions
- Dramatically reduces re-render count for responsive components

### When to Apply

- Window/viewport size checks for responsive layouts
- Scroll position thresholds (past header, near bottom, etc.)
- Count-based conditions (isEmpty, hasItems, isOverLimit)
- Any subscription where you only care about crossing a boundary

### When to Skip

- You need the actual value for calculations or display (e.g., showing "Width: 1024px")
- Multiple thresholds need checking against the same value
- The continuous value is already throttled/debounced at the source

### Expected Impact

Reduces re-render frequency by only updating when the derived boolean crosses its threshold. Instead of re-rendering on every pixel change during window resize (potentially hundreds of times per second), the component only re-renders when crossing the breakpoint boundary.

### See Also

- `rerender-dependencies.md` - Narrow effect dependencies to derived values
- `rerender-transitions.md` - Use transitions for non-urgent updates from continuous sources
