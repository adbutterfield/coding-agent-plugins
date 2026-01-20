---
title: Per-Request Deduplication with React.cache()
impact: MEDIUM
---

## Per-Request Deduplication with React.cache()

> **Review:** `cache(async (` with inline object parameters like `{ uid: number }`
> **Writing:** When creating cached server functions that accept parameters
> **Technique:** Use primitive arguments instead of objects for React.cache() functions

Use `React.cache()` for server-side request deduplication. Authentication and database queries benefit most. Within a single request, multiple calls to a cached function execute the query only once. However, `React.cache()` uses shallow equality (`Object.is`) - inline objects create new references each call, preventing cache hits.

### Fail

```typescript
// ❌ Inline objects always create new references, causing cache misses
const getUser = cache(async (params: { uid: number }) => {
  return await db.user.findUnique({ where: { id: params.uid } })
})

// Each call creates new object, never hits cache
getUser({ uid: 1 })
getUser({ uid: 1 })  // Cache miss, runs query again
```

### Pass

```typescript
// ✅ Primitive args use value equality for reliable cache hits
const getUser = cache(async (uid: number) => {
  return await db.user.findUnique({ where: { id: uid } })
})

// Primitive args use value equality
getUser(1)
getUser(1)  // Cache hit, returns cached result
```

If you must pass objects, pass the same reference:

```typescript
// ✅ Same object reference will cache correctly
const params = { uid: 1 }
getUser(params)  // Query runs
getUser(params)  // Cache hit (same reference)
```

### Why This Matters

- `React.cache()` deduplicates expensive operations (DB queries, auth checks) within a single request
- Object arguments defeat caching due to reference inequality on each call
- Primitive arguments enable automatic deduplication across your component tree

### When to Apply

- Database queries in Server Components
- Authentication checks called from multiple components
- Heavy computations that may be called multiple times per request
- Any non-fetch async work (Next.js already memoizes `fetch` calls)

### When to Skip

- When using Next.js `fetch()` which has built-in memoization
- Client-side code (React.cache is server-only)
- Functions that intentionally need fresh data on each call
- When the cached function is only ever called once per request

### Expected Impact

Deduplicates identical fetches within a single request, reducing redundant database and API calls. This optimization can eliminate multiple unnecessary queries per request when the same cached function is called from different parts of your component tree.

### See Also

- `server-cache-lru.md` - Cross-request caching for data shared between sequential requests
- `server-parallel-fetching.md` - Parallel data fetching with component composition
