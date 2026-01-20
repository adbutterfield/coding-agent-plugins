---
title: Optimize SVG Precision
impact: LOW
---

## Optimize SVG Precision

> **Review:** SVG paths with coordinates like `10.293847` (6+ decimal places)
> **Writing:** Adding SVG icons or illustrations to the codebase
> **Technique:** Reduce coordinate precision to 1-2 decimal places using SVGO

Reduce SVG coordinate precision to decrease file size. Most SVGs don't need more than 1-2 decimal places of precision, especially for icons and UI elements.

### Fail

```svg
<!-- ❌ Excessive precision wastes bytes -->
<path d="M 10.293847 20.847362 L 30.938472 40.192837" />
```

### Pass

```svg
<!-- ✅ Reduced precision, visually identical -->
<path d="M 10.3 20.8 L 30.9 40.2" />
```

### Automate with SVGO

```bash
# Optimize single file
npx svgo --precision=1 --multipass icon.svg

# Optimize entire directory
npx svgo --precision=1 --multipass -f ./icons

# Add to package.json scripts
# "optimize:svg": "svgo --precision=1 --multipass -f ./src/assets/icons"
```

### Why This Matters

- Each unnecessary decimal adds bytes across every coordinate
- A typical icon might have 50-200 coordinate values
- 1-2 decimal places is visually indistinguishable for icons at typical sizes
- Smaller SVGs = faster downloads and smaller bundle size

### When to Apply

- Icon libraries
- Logo assets
- UI illustrations
- Any SVG added to the codebase from design tools (Figma, Illustrator export with high precision)

### When to Skip

- Very large viewBox sizes where precision matters (detailed maps, technical drawings)
- SVGs that will be displayed at extreme zoom levels
- Animations that require smooth curves at sub-pixel level
- When precision reduction causes visible artifacts (test first)

### Expected Impact

Reduces SVG file size by 10-30% without visible quality loss. This translates to smaller bundle sizes and faster downloads, with the savings multiplying across icon libraries containing dozens or hundreds of SVGs.

### See Also

- `rendering-hoist-jsx.md` - Hoist optimized SVGs as static elements
- `rendering-animate-svg-wrapper.md` - Animate SVGs with wrapper divs
