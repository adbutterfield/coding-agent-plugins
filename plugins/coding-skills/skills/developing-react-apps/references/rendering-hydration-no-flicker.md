---
title: Prevent Hydration Mismatch Without Flickering
impact: MEDIUM
---

## Prevent Hydration Mismatch Without Flickering

> **Review:** `localStorage`/`sessionStorage` accessed directly in render or in `useEffect` to set initial state
> **Writing:** Need to render content based on client-side storage (localStorage, cookies, preferences)
> **Technique:** Inject a synchronous script that updates DOM before React hydrates

When rendering content that depends on client-side storage, avoid both SSR breakage and post-hydration flickering by injecting a synchronous script that updates the DOM before React hydrates.

### Fail

```tsx
// ❌ Breaks SSR: localStorage undefined on server
function ThemeWrapper({ children }: { children: ReactNode }) {
  const theme = localStorage.getItem('theme') || 'light'

  return <div className={theme}>{children}</div>
}
```

```tsx
// ❌ Causes flicker: renders default, then updates after hydration
function ThemeWrapper({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState('light')

  useEffect(() => {
    const stored = localStorage.getItem('theme')
    if (stored) setTheme(stored)
  }, [])

  return <div className={theme}>{children}</div>
}
```

### Pass

```tsx
// ✅ No flicker, no hydration mismatch: script runs before React hydrates
function ThemeWrapper({ children }: { children: ReactNode }) {
  return (
    <>
      <div id="theme-wrapper">{children}</div>
      <script
        dangerouslySetInnerHTML={{
          __html: `
            (function() {
              try {
                var theme = localStorage.getItem('theme') || 'light';
                var el = document.getElementById('theme-wrapper');
                if (el) el.className = theme;
              } catch (e) {}
            })();
          `,
        }}
      />
    </>
  )
}
```

### Why This Matters

- Direct `localStorage` access breaks SSR completely
- `useEffect` approach causes visible flash—user sees default then correct value
- Inline script executes synchronously before paint, ensuring correct DOM from first render

### When to Apply

- Theme toggles (dark mode)
- User preferences stored in localStorage/cookies
- Authentication states affecting initial render
- Any client-only data that should render immediately without flashing

### When to Skip

- Data that can be fetched server-side (use SSR data fetching instead)
- Non-visual preferences that don't affect initial render
- Content where brief loading state is acceptable

### Expected Impact

Eliminates visual flicker (FOUC - Flash of Unstyled Content) when rendering client-only data like themes or user preferences. Users see the correct theme or preference state from the very first paint, rather than briefly seeing a default value before it updates.

### See Also

- `rendering-ssr-client.md` - Client vs server rendering decisions
