---
title: Batch DOM CSS Changes
impact: MEDIUM
---

## Batch DOM CSS Changes

> **Review:** Look for `offsetWidth`, `offsetHeight`, `getBoundingClientRect()`, or `getComputedStyle()` between style assignments
> **Writing:** When modifying multiple element styles and then reading layout properties
> **Technique:** Group all style writes together, then perform reads after all writes complete

Interleaving style writes with layout reads forces synchronous reflows. Each read triggers the browser to recalculate layout, causing expensive repaints.

### Fail

```typescript
// ❌ Interleaved reads and writes force multiple reflows
function updateElementStyles(element: HTMLElement) {
  element.style.width = '100px'
  const width = element.offsetWidth  // Forces reflow
  element.style.height = '200px'
  const height = element.offsetHeight  // Forces another reflow
}
```

### Pass

```typescript
// ✅ Batch all writes together, then read once (single reflow)
function updateElementStyles(element: HTMLElement) {
  element.style.width = '100px'
  element.style.height = '200px'
  element.style.backgroundColor = 'blue'
  element.style.border = '1px solid black'

  const { width, height } = element.getBoundingClientRect()
}
```

### Better: Use CSS Classes

```css
.highlighted-box {
  width: 100px;
  height: 200px;
  background-color: blue;
  border: 1px solid black;
}
```

```typescript
// ✅ CSS classes are cached and easier to maintain
function updateElementStyles(element: HTMLElement) {
  element.classList.add('highlighted-box')

  const { width, height } = element.getBoundingClientRect()
}
```

### Why This Matters

- Each layout read between style writes triggers a synchronous reflow
- CSS files are cached by the browser; inline styles are not
- CSS classes provide better separation of concerns and maintainability

### When to Apply

- Animating or transitioning multiple element properties
- Updating styles based on calculated dimensions
- Batch operations on multiple DOM elements

### When to Skip

- Single style change with no subsequent layout read
- Using CSS animations or transitions (browser batches these automatically)
- When `requestAnimationFrame` is already batching your updates

### Expected Impact

Reduces layout thrashing by batching DOM reads and writes together. This minimizes forced synchronous reflows and repaints, which are among the most expensive browser operations.

### See Also

- `js-cache-property-access.md` - Cache repeated property lookups in loops
