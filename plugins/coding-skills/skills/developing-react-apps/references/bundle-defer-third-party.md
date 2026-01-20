---
title: Defer Non-Critical Third-Party Libraries
impact: MEDIUM
---

## Defer Non-Critical Third-Party Libraries

> **Review:** Static `import` of analytics, logging, or error tracking in layout/root
> **Writing:** Adding analytics, monitoring, or tracking to an application
> **Technique:** Use `next/dynamic` with `ssr: false` to load after hydration

Analytics, logging, and error tracking libraries don't block user interaction. Loading them synchronously delays hydration and increases Time to Interactive without user-facing benefit.

### Fail

```tsx
// ❌ Analytics blocks initial bundle and hydration
import { Analytics } from '@vercel/analytics/react'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

### Pass

```tsx
// ✅ Analytics loads after hydration completes
import dynamic from 'next/dynamic'

const Analytics = dynamic(
  () => import('@vercel/analytics/react').then(m => m.Analytics),
  { ssr: false }
)

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

### Why This Matters

- Third-party scripts compete for main thread during critical rendering
- Analytics data is still captured; it just initializes slightly later
- Users get faster interactivity without losing tracking capabilities

### When to Apply

- Analytics libraries (`@vercel/analytics`, Google Analytics, Mixpanel, Amplitude)
- Error tracking (Sentry, Bugsnag, LogRocket)
- Customer support widgets (Intercom, Zendesk, Crisp)
- A/B testing and feature flag clients that don't affect initial render
- Any third-party library that doesn't produce visible UI on initial load

### When to Skip

- Error tracking that must capture errors during initial render
- Feature flag clients that determine what content to show above the fold
- Authentication providers needed before rendering protected content
- Third-party widgets that are part of the above-the-fold experience

### Expected Impact

Moves third-party scripts out of the critical rendering path, loading them after hydration completes. This reduces Time to Interactive without losing tracking capabilities, as analytics and monitoring still capture all relevant data once initialized.

### See Also

- `bundle-dynamic-imports.md` - Dynamic imports for heavy first-party components
- `bundle-preload.md` - Preload libraries based on user intent
