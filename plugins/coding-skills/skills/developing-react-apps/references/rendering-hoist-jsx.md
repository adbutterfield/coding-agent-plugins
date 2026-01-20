---
title: Hoist Static JSX Elements
impact: LOW
---

## Hoist Static JSX Elements

> **Review:** Static JSX (no props/state) defined inside component functions
> **Writing:** Creating loading skeletons, placeholder content, or static decorative elements
> **Technique:** Extract static JSX to module-level constants outside the component

Extract static JSX elements outside components to avoid recreating them on every render. This is especially impactful for large SVG nodes and static markup used in frequently-rendered components.

### Fail

```tsx
// ❌ Creates new LoadingSkeleton element on every Container render
function LoadingSkeleton() {
  return <div className="animate-pulse h-20 bg-gray-200" />
}

function Container() {
  return (
    <div>
      {loading && <LoadingSkeleton />}
    </div>
  )
}
```

### Pass

```tsx
// ✅ Reuses same element reference across all renders
const loadingSkeleton = (
  <div className="animate-pulse h-20 bg-gray-200" />
)

function Container() {
  return (
    <div>
      {loading && loadingSkeleton}
    </div>
  )
}
```

### Why This Matters

- React compares element references for reconciliation
- Same reference = React skips diffing that subtree entirely
- Significant savings for complex static SVGs or deeply nested markup

### When to Apply

- Loading skeletons and placeholders
- Static icons and decorative SVGs
- Empty state illustrations
- Repeated static markup in lists
- Any JSX that never changes based on props or state

### When to Skip

- Elements that depend on props, state, or context
- Elements that need different keys for list rendering
- When using React Compiler (it auto-hoists static JSX)
- Very simple elements where the optimization overhead isn't worth it

### Expected Impact

Avoids re-creating static JSX elements on every render, reducing garbage collection pressure and improving React's reconciliation speed. When React sees the same element reference, it skips diffing that entire subtree, which is especially beneficial for complex static SVGs or deeply nested markup.

### See Also

- `rendering-animate-svg-wrapper.md` - Animate hoisted SVGs properly
- `rendering-svg-precision.md` - Optimize SVG content before hoisting
