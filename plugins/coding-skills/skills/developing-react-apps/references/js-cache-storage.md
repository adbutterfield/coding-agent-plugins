---
title: Cache Storage API Calls
impact: LOW-MEDIUM
---

## Cache Storage API Calls

> **Review:** Look for `localStorage.getItem`, `sessionStorage.getItem`, or `document.cookie` access in frequently called code
> **Writing:** When reading browser storage repeatedly during render or in loops
> **Technique:** Cache storage reads in a module-level Map, keep it in sync on writes

`localStorage`, `sessionStorage`, and `document.cookie` are synchronous and expensive. Cache reads in memory to avoid repeated I/O.

### Fail

```typescript
// ❌ Reads storage on every call
function getTheme() {
  return localStorage.getItem('theme') ?? 'light'
}
// Called 10 times = 10 storage reads
```

### Pass

```typescript
// ✅ Map cache - read storage once, then use memory
const storageCache = new Map<string, string | null>()

function getLocalStorage(key: string) {
  if (!storageCache.has(key)) {
    storageCache.set(key, localStorage.getItem(key))
  }
  return storageCache.get(key)
}

function setLocalStorage(key: string, value: string) {
  localStorage.setItem(key, value)
  storageCache.set(key, value)  // keep cache in sync
}
```

### Cookie Caching

```typescript
// ✅ Parse cookies once, cache the result
let cookieCache: Record<string, string> | null = null

function getCookie(name: string) {
  if (!cookieCache) {
    cookieCache = Object.fromEntries(
      document.cookie.split('; ').map(c => c.split('='))
    )
  }
  return cookieCache[name]
}
```

### Invalidate on External Changes

```typescript
// ✅ Handle storage changes from other tabs
window.addEventListener('storage', (e) => {
  if (e.key) storageCache.delete(e.key)
})

// ✅ Clear cache when tab becomes visible (may have stale data)
document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'visible') {
    storageCache.clear()
  }
})
```

### Why This Matters

- Storage APIs are synchronous and block the main thread
- Module-level Map works everywhere: utilities, event handlers, not just React components
- Storage reads can be 10-100x slower than memory access

### When to Apply

- Theme, locale, or preference lookups called during render
- Authentication checks using cookies
- Any storage access in hot paths or loops

### When to Skip

- Single read at app initialization
- When data must always be fresh (check storage directly)
- Server-side rendering (no storage APIs available)

### Expected Impact

Reduces expensive localStorage, sessionStorage, and cookie I/O by caching reads in memory. Storage API calls can be 10-100x slower than memory access, so caching eliminates this overhead on repeated reads.

### See Also

- `js-cache-function-results.md` - Cache expensive function return values
