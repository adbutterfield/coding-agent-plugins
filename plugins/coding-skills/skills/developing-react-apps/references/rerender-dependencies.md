---
title: Narrow Effect Dependencies
impact: LOW
---

## Narrow Effect Dependencies

> **Review:** `useEffect` with object in dependency array when only a property is used inside
> **Writing:** Creating an effect that reacts to changes in specific values
> **Technique:** Use primitive properties instead of whole objects in dependency arrays

Specifying primitive dependencies instead of objects minimizes effect re-runs. Objects create new references on every render, causing effects to run even when the relevant data hasn't changed.

### Fail

```tsx
// ❌ Re-runs on any user field change (name, email, avatar, etc.)
useEffect(() => {
  console.log(user.id)
}, [user])
```

```tsx
// ❌ Runs on width=767, 766, 765... every pixel change
useEffect(() => {
  if (width < 768) {
    enableMobileMode()
  }
}, [width])
```

### Pass

```tsx
// ✅ Re-runs only when id changes
useEffect(() => {
  console.log(user.id)
}, [user.id])
```

```tsx
// ✅ Runs only on boolean transition (desktop <-> mobile)
const isMobile = width < 768
useEffect(() => {
  if (isMobile) {
    enableMobileMode()
  }
}, [isMobile])
```

### Why This Matters

- Effects with narrow dependencies run less frequently
- Derived boolean values collapse many value changes into fewer state transitions
- Reduces unnecessary side effect execution and improves performance

### When to Apply

- Effects that only use specific properties of an object
- Effects that respond to threshold crossings (width < 768, count > 0, etc.)
- Any dependency array where a subset of the data is actually used

### When to Skip

- The effect genuinely needs to run when any part of the object changes
- The object is stable (comes from context, useMemo, or external store)
- Destructuring would make the code less readable without meaningful benefit

### Expected Impact

Minimizes effect re-runs by using stable primitive values instead of objects that change identity on every render. Effects with narrowed dependencies execute only when their specific values change, reducing unnecessary side effect execution and improving overall application responsiveness.

### See Also

- `rerender-derived-state.md` - Subscribe to derived state instead of continuous values
- `rerender-defer-reads.md` - Defer reads to avoid unnecessary subscriptions
