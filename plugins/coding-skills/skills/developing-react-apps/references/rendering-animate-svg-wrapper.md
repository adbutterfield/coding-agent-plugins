---
title: Animate SVG Wrapper Instead of SVG Element
impact: LOW
---

## Animate SVG Wrapper Instead of SVG Element

> **Review:** `<svg` with `animate-`, `transition`, `transform` classes directly on the SVG element
> **Writing:** Creating animated SVG icons, spinners, or any SVG with CSS animations
> **Technique:** Wrap SVG in a `<div>` and apply animation classes to the wrapper

Many browsers lack hardware acceleration for CSS3 animations on SVG elements. Wrapping the SVG in a div enables GPU-accelerated animations for smoother performance.

### Fail

```tsx
// ❌ No hardware acceleration - animation runs on CPU
function LoadingSpinner() {
  return (
    <svg
      className="animate-spin"
      width="24"
      height="24"
      viewBox="0 0 24 24"
    >
      <circle cx="12" cy="12" r="10" stroke="currentColor" />
    </svg>
  )
}
```

### Pass

```tsx
// ✅ Hardware accelerated - animation runs on GPU
function LoadingSpinner() {
  return (
    <div className="animate-spin">
      <svg
        width="24"
        height="24"
        viewBox="0 0 24 24"
      >
        <circle cx="12" cy="12" r="10" stroke="currentColor" />
      </svg>
    </div>
  )
}
```

### Why This Matters

- CSS transforms on SVG elements may not trigger GPU compositing in some browsers
- Wrapper divs reliably use hardware acceleration for `transform`, `opacity`, `translate`, `scale`, `rotate`
- Smoother 60fps animations with less CPU usage

### When to Apply

- Spinner/loading icons with rotation animations
- SVG icons with hover transitions
- Any SVG using CSS transforms or transitions
- Animated illustrations or graphics

### When to Skip

- SVG-internal animations using SMIL or CSS animations on SVG child elements (paths, circles)
- When the SVG needs to fill its container without a wrapper affecting layout
- Static SVGs with no animation

### Expected Impact

Enables hardware-accelerated (GPU) animations, achieving smooth 60fps instead of janky main-thread animations. This results in noticeably smoother visual performance, especially for continuously running animations like spinners or hover effects.

### See Also

- `rendering-svg-precision.md` - Optimize SVG file size
- `rendering-hoist-jsx.md` - Hoist static SVG elements
