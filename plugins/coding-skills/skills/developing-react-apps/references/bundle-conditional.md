---
title: Conditional Module Loading
impact: HIGH
---

## Conditional Module Loading

> **Review:** Top-level `import` of large data files or modules used conditionally
> **Writing:** Adding a feature that requires large data only when activated
> **Technique:** Use dynamic `import()` inside `useEffect` when the condition is true

Large data files and modules should only load when their feature is actually activated. Top-level imports bundle everything immediately, even if the user never enables the feature.

### Fail

```tsx
// ❌ Animation frames bundled immediately, even if AnimationPlayer is never used
import { frames } from './animation-frames.js'

function AnimationPlayer({ enabled }: { enabled: boolean }) {
  if (!enabled) return null
  return <Canvas frames={frames} />
}
```

### Pass

```tsx
// ✅ Animation frames load only when feature is enabled
function AnimationPlayer({ enabled, setEnabled }: { enabled: boolean; setEnabled: React.Dispatch<React.SetStateAction<boolean>> }) {
  const [frames, setFrames] = useState<Frame[] | null>(null)

  useEffect(() => {
    if (enabled && !frames && typeof window !== 'undefined') {
      import('./animation-frames.js')
        .then(mod => setFrames(mod.frames))
        .catch(() => setEnabled(false))
    }
  }, [enabled, frames, setEnabled])

  if (!frames) return <Skeleton />
  return <Canvas frames={frames} />
}
```

### Why This Matters

- The `typeof window !== 'undefined'` check prevents bundling for SSR
- Users only download data for features they actually use
- Server bundle stays lean, improving build speed and cold start times

### When to Apply

- Large JSON data files (animation frames, map data, locale dictionaries)
- Feature-flagged functionality with heavy dependencies
- Optional advanced features most users don't enable
- Premium features behind authentication or subscription checks

### When to Skip

- Data that is always needed regardless of user actions
- Small modules where the loading overhead exceeds the bundle savings
- Critical path data where a loading state would hurt UX
- Modules needed synchronously before render

### Expected Impact

Loads large modules only when the feature is actually activated, reducing initial bundle size. Users only download data for features they use, keeping the main bundle lean and improving both build speed and cold start times.

### See Also

- `bundle-dynamic-imports.md` - Use next/dynamic for component-level code splitting
- `bundle-preload.md` - Preload conditional modules based on user intent
