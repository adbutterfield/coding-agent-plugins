---
title: Version and Minimize localStorage Data
impact: MEDIUM
---

## Version and Minimize localStorage Data

> **Review:** `localStorage.setItem` without version prefix or try-catch, storing full objects
> **Writing:** When persisting user preferences or cached data to localStorage
> **Technique:** Add version prefix to keys, store only needed fields, wrap in try-catch

Add version prefix to keys and store only needed fields. Prevents schema conflicts and accidental storage of sensitive data.

### Fail

```typescript
// ❌ No version, stores everything, no error handling
localStorage.setItem('userConfig', JSON.stringify(fullUserObject))
const data = localStorage.getItem('userConfig')
```

### Pass

```typescript
// ✅ Versioned keys with error handling
const VERSION = 'v2'

function saveConfig(config: { theme: string; language: string }) {
  try {
    localStorage.setItem(`userConfig:${VERSION}`, JSON.stringify(config))
  } catch {
    // Throws in incognito/private browsing, quota exceeded, or disabled
  }
}

function loadConfig() {
  try {
    const data = localStorage.getItem(`userConfig:${VERSION}`)
    return data ? JSON.parse(data) : null
  } catch {
    return null
  }
}

// Migration from v1 to v2
function migrate() {
  try {
    const v1 = localStorage.getItem('userConfig:v1')
    if (v1) {
      const old = JSON.parse(v1)
      saveConfig({ theme: old.darkMode ? 'dark' : 'light', language: old.lang })
      localStorage.removeItem('userConfig:v1')
    }
  } catch {}
}
```

```typescript
// ✅ Store minimal fields from server responses
// User object has 20+ fields, only store what UI needs
function cachePrefs(user: FullUser) {
  try {
    localStorage.setItem('prefs:v1', JSON.stringify({
      theme: user.preferences.theme,
      notifications: user.preferences.notifications
    }))
  } catch {}
}
```

### Why This Matters

- Schema evolution via versioning prevents data corruption when format changes
- Reduced storage size improves performance and avoids quota limits
- Prevents accidentally storing tokens, PII, or internal flags

### When to Apply

- Any localStorage/sessionStorage usage for user preferences
- Caching server responses client-side
- Persisting UI state across sessions

### When to Skip

- Temporary storage that gets cleared on logout
- When using a storage abstraction library that handles versioning
- IndexedDB usage where schema migrations are built-in

### Expected Impact

Versioned keys prevent schema conflicts and data corruption when storage format changes between versions. Users get clean migrations instead of parse errors or stale data causing UI bugs after updates.

### See Also

- `js-cache-storage.md` - Caching patterns for computed values
