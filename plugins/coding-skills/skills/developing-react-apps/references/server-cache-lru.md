---
title: Cross-Request LRU Caching
impact: HIGH
---

## Cross-Request LRU Caching

> **Review:** Repeated database queries without caching, especially for data accessed across sequential requests
> **Writing:** When implementing data fetching for frequently accessed, rarely changing data
> **Technique:** Use LRU cache to share data across requests instead of hitting the database each time

`React.cache()` only works within one request. For data shared across sequential requests (user clicks button A then button B), use an LRU cache. This is especially effective with Vercel's Fluid Compute where multiple concurrent requests share the same function instance.

### Fail

```typescript
// ❌ Database hit on every request, even for the same user
export async function getUser(id: string) {
  const user = await db.user.findUnique({ where: { id } })
  return user
}

// Request 1: DB query
// Request 2: DB query again for same user
```

### Pass

```typescript
// ✅ LRU cache shares data across sequential requests
import { LRUCache } from 'lru-cache'

const cache = new LRUCache<string, any>({
  max: 1000,
  ttl: 5 * 60 * 1000  // 5 minutes
})

export async function getUser(id: string) {
  const cached = cache.get(id)
  if (cached) return cached

  const user = await db.user.findUnique({ where: { id } })
  cache.set(id, user)
  return user
}

// Request 1: DB query, result cached
// Request 2: cache hit, no DB query
```

### Why This Matters

- Reduces database load for frequently accessed data
- Improves response times for sequential user actions
- With Vercel's Fluid Compute, cache persists across requests without external storage

### When to Apply

- Sequential user actions hitting multiple endpoints needing the same data
- Frequently accessed reference data (user profiles, settings, feature flags)
- Read-heavy workloads with data that changes infrequently
- Vercel Fluid Compute environments where function instances persist

### When to Skip

- Traditional serverless with isolated invocations (use Redis instead)
- Data that must be real-time fresh on every request
- Highly personalized data that varies per user with no repeat access
- When cache invalidation complexity outweighs benefits

### Expected Impact

Caches expensive computations and database queries across requests, significantly reducing response times for repeated queries. With proper TTL configuration, this can reduce database load by 80-90% for frequently accessed data and cut response times from hundreds of milliseconds to single-digit milliseconds on cache hits.

### See Also

- `server-cache-react.md` - Per-request deduplication with React.cache()
- `js-cache-function-results.md` - Memoizing expensive function results
