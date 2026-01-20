---
title: Avoid Barrel File Imports
impact: CRITICAL
---

## Avoid Barrel File Imports

> **Review:** `import { ... } from 'lucide-react'` or `from '@mui/material'` without deep paths
> **Writing:** Importing icons, components, or utilities from a library's main entry point
> **Technique:** Use direct imports from source files or configure `optimizePackageImports`

Barrel files re-export thousands of modules from a single entry point. Popular libraries can have up to 10,000 re-exports, adding 200-800ms import cost per cold start and significantly slowing builds.

### Fail

```tsx
// ❌ Imports entire library through barrel file
import { Check, X, Menu } from 'lucide-react'
// Loads 1,583 modules, takes ~2.8s extra in dev
// Runtime cost: 200-800ms on every cold start

import { Button, TextField } from '@mui/material'
// Loads 2,225 modules, takes ~4.2s extra in dev
```

### Pass

```tsx
// ✅ Direct imports load only what you need
import Check from 'lucide-react/dist/esm/icons/check'
import X from 'lucide-react/dist/esm/icons/x'
import Menu from 'lucide-react/dist/esm/icons/menu'
// Loads only 3 modules (~2KB vs ~1MB)

import Button from '@mui/material/Button'
import TextField from '@mui/material/TextField'
// Loads only what you use
```

**Alternative (Next.js 13.5+):**

```js
// ✅ Configure automatic optimization in next.config.js
module.exports = {
  experimental: {
    optimizePackageImports: ['lucide-react', '@mui/material']
  }
}

// Then you can keep the ergonomic barrel imports:
import { Check, X, Menu } from 'lucide-react'
// Automatically transformed to direct imports at build time
```

### Why This Matters

- Tree-shaking cannot help when libraries are marked as external (not bundled)
- Bundling libraries to enable tree-shaking makes builds substantially slower
- Direct imports provide 15-70% faster dev boot, 28% faster builds, 40% faster cold starts

### When to Apply

- Importing from icon libraries (`lucide-react`, `@mui/icons-material`, `@tabler/icons-react`, `react-icons`)
- Importing from component libraries (`@mui/material`, `@headlessui/react`, `@radix-ui/react-*`)
- Importing from utility libraries (`lodash`, `ramda`, `date-fns`, `rxjs`, `react-use`)

### When to Skip

- Using Next.js 13.5+ with `optimizePackageImports` configured for the library
- Libraries that provide properly tree-shakeable ESM exports with no side effects
- Internal application barrel files with only a few exports

### Expected Impact

Expect a 200-800ms reduction in import cost per cold start and significantly faster builds. Direct imports can provide 15-70% faster dev boot times, 28% faster production builds, and 40% faster cold starts compared to barrel file imports.

### See Also

- `bundle-dynamic-imports.md` - Lazy-load heavy components with next/dynamic
- `bundle-defer-third-party.md` - Defer non-critical libraries until after hydration

Reference: [How we optimized package imports in Next.js](https://vercel.com/blog/how-we-optimized-package-imports-in-next-js)
