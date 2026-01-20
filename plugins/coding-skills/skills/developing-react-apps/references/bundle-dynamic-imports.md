---
title: Dynamic Imports for Heavy Components
impact: CRITICAL
---

## Dynamic Imports for Heavy Components

> **Review:** Static `import` of Monaco, charts, rich editors, or components >50KB
> **Writing:** Adding a large component that isn't needed on initial page load
> **Technique:** Use `next/dynamic` with `ssr: false` to lazy-load on demand

Heavy components like code editors, charting libraries, and rich text editors can add hundreds of KB to your main bundle. This directly affects Time to Interactive (TTI) and Largest Contentful Paint (LCP).

### Fail

```tsx
// ❌ Monaco bundles with main chunk (~300KB)
import { MonacoEditor } from './monaco-editor'

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```

### Pass

```tsx
// ✅ Monaco loads on demand when component renders
import dynamic from 'next/dynamic'

const MonacoEditor = dynamic(
  () => import('./monaco-editor').then(m => m.MonacoEditor),
  { ssr: false }
)

function CodePanel({ code }: { code: string }) {
  return <MonacoEditor value={code} />
}
```

### Why This Matters

- Main bundle size directly impacts initial page load time
- Users pay the cost of downloading code they may never use
- Code splitting moves heavy dependencies out of the critical path

### When to Apply

- Code editors (Monaco, CodeMirror, Ace)
- Charting libraries (Chart.js, Recharts, D3 visualizations)
- Rich text editors (TipTap, Slate, Draft.js)
- PDF viewers, video players, or complex interactive widgets
- Any component over 50KB that isn't visible above the fold

### When to Skip

- Component is critical for above-the-fold content
- Component is needed immediately on page load
- The loading state would cause layout shift in a prominent area
- Server-side rendering is required for SEO on that component

### Expected Impact

Directly improves Time to Interactive (TTI) and Largest Contentful Paint (LCP) by removing hundreds of kilobytes from the initial bundle. Users experience faster page loads and interactivity since heavy components like editors and charts load only when needed.

### See Also

- `bundle-preload.md` - Preload dynamic imports based on user intent
- `bundle-conditional.md` - Conditionally load modules based on feature flags
- `bundle-defer-third-party.md` - Defer analytics and tracking libraries
