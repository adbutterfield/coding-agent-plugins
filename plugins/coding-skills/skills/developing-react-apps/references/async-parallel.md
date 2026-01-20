---
title: Promise.all() for Independent Operations
impact: CRITICAL
---

## Promise.all() for Independent Operations

> **Review:** Sequential `await` statements on independent async calls
> **Writing:** Need results from multiple independent async operations
> **Technique:** Wrap in `Promise.all([...])`

Sequential awaits on independent operations create request waterfalls, adding 2-10× latency.

### Fail

```typescript
// ❌ Sequential: 3 round trips, each waits for previous
const user = await fetchUser()
const posts = await fetchPosts()
const comments = await fetchComments()
```

### Pass

```typescript
// ✅ Parallel: 1 round trip, all execute concurrently
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
```

### Why This Matters

- Each `await` blocks until complete, turning parallel-capable operations into a waterfall
- Network latency compounds: 3 × 100ms = 300ms sequential vs 100ms parallel

### When to Apply

- Multiple async calls that don't depend on each other's results
- Data fetching in React components or API routes
- Any I/O-bound operations (database, network, file system)

### When to Skip

- Operations that depend on previous results (see `async-dependencies.md`)
- Single async operation (no benefit from parallelization)

### Expected Impact

2-10x latency reduction for independent async operations. The improvement scales with the number of parallel operations and their individual latencies.

### See Also

- `async-defer-await.md` - Defer await until needed
- `async-dependencies.md` - Parallel with partial dependencies
