---
title: Preload Based on User Intent
impact: MEDIUM
---

## Preload Based on User Intent

> **Review:** Dynamic imports without preload triggers on hover, focus, or intent signals
> **Writing:** Adding a button or link that triggers loading a heavy component
> **Technique:** Call `import()` on `onMouseEnter`/`onFocus` to start loading before click

Preloading heavy bundles before they're needed reduces perceived latency. When users hover or focus on interactive elements, start fetching the code they'll need on click.

### Fail

```tsx
// ❌ Editor loads only after click, causing noticeable delay
function EditorButton({ onClick }: { onClick: () => void }) {
  return (
    <button onClick={onClick}>
      Open Editor
    </button>
  )
}
```

### Pass

```tsx
// ✅ Editor preloads on hover/focus, ready by click time
function EditorButton({ onClick }: { onClick: () => void }) {
  const preload = () => {
    if (typeof window !== 'undefined') {
      void import('./monaco-editor')
    }
  }

  return (
    <button
      onMouseEnter={preload}
      onFocus={preload}
      onClick={onClick}
    >
      Open Editor
    </button>
  )
}
```

**Preload when feature flag is enabled:**

```tsx
// ✅ Preload immediately when user has access to feature
function FlagsProvider({ children, flags }: Props) {
  useEffect(() => {
    if (flags.editorEnabled && typeof window !== 'undefined') {
      void import('./monaco-editor').then(mod => mod.init())
    }
  }, [flags.editorEnabled])

  return <FlagsContext.Provider value={flags}>
    {children}
  </FlagsContext.Provider>
}
```

### Why This Matters

- The `typeof window !== 'undefined'` check prevents bundling for SSR
- Average hover-to-click time is 300-500ms, enough to fetch many modules
- Perceived performance improves even if total load time is unchanged

### When to Apply

- Buttons that open modals with heavy content (editors, charts, forms)
- Navigation links to routes with large bundles
- Feature flags that unlock heavy functionality
- Tabs or accordions containing complex components

### When to Skip

- Components that are already small (<10KB)
- Touch-only interfaces where hover isn't available
- High-frequency interactions where preload would waste bandwidth
- When the module is already likely cached from previous visits

### Expected Impact

Reduces perceived latency by loading code before the user clicks. Since the average hover-to-click time is 300-500ms, preloading can fetch many modules during this window, making interactions feel instant even when loading heavy components.

### See Also

- `bundle-dynamic-imports.md` - Set up dynamic imports that can be preloaded
- `bundle-conditional.md` - Load modules conditionally based on state
