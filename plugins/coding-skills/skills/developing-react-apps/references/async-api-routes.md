---
title: Prevent Waterfall Chains in API Routes
impact: CRITICAL
---

## Prevent Waterfall Chains in API Routes

> **Review:** Sequential `await` calls in API route handlers or Server Actions
> **Writing:** API route or Server Action that fetches multiple pieces of data
> **Technique:** Start promises immediately, await them together or when their data is needed

Adds 2-10x latency by running independent fetches sequentially instead of in parallel.

### Fail

```typescript
// ❌ config waits for auth, data waits for both - waterfall chain
export async function GET(request: Request) {
  const session = await auth()
  const config = await fetchConfig()
  const data = await fetchData(session.user.id)
  return Response.json({ data, config })
}
```

### Pass

```typescript
// ✅ auth and config start immediately, data waits only for session
export async function GET(request: Request) {
  const sessionPromise = auth()
  const configPromise = fetchConfig()
  const session = await sessionPromise
  const [config, data] = await Promise.all([
    configPromise,
    fetchData(session.user.id)
  ])
  return Response.json({ data, config })
}
```

### Why This Matters

- API routes and Server Actions run on the server where network calls are cheap to start
- Sequential awaits create artificial bottlenecks
- Starting promises early and awaiting later maximizes parallelism

### When to Apply

- API route handlers with multiple data fetches
- Server Actions that aggregate data from multiple sources
- Any server-side code with independent async operations

### When to Skip

- Operations have strict ordering requirements (e.g., create then update)
- Single async operation in the route
- Operations are so fast that parallelization overhead isn't worth it
- Error handling requires specific ordering (fail fast on auth before other fetches)

### Expected Impact

2-10x latency reduction in API route response times. Server-side parallelization is particularly effective because network calls are cheap to start, and the improvement directly impacts user-perceived performance.

### See Also

- `async-dependencies.md` - For complex dependency chains, use better-all
- `async-parallel.md` - General parallelization patterns
- `async-defer-await.md` - Defer operations that may not be needed
