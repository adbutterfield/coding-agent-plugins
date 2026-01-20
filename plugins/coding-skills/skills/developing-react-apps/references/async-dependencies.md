---
title: Dependency-Based Parallelization
impact: CRITICAL
---

## Dependency-Based Parallelization

> **Review:** `Promise.all` followed by an await that uses results from the Promise.all
> **Writing:** Multiple async operations where some depend on others' results
> **Technique:** Use `better-all` to start each task at the earliest possible moment

Adds 2-10x latency when independent operations wait unnecessarily for dependent ones.

### Fail

```typescript
// ❌ profile waits for config unnecessarily
const [user, config] = await Promise.all([
  fetchUser(),
  fetchConfig()
])
const profile = await fetchProfile(user.id)
```

### Pass

```typescript
// ✅ config and profile run in parallel, profile starts as soon as user resolves
import { all } from 'better-all'

const { user, config, profile } = await all({
  async user() { return fetchUser() },
  async config() { return fetchConfig() },
  async profile() {
    return fetchProfile((await this.$.user).id)
  }
})
```

### Why This Matters

- `Promise.all` forces all operations to complete before any dependent work starts
- `better-all` automatically tracks dependencies and starts each task immediately when its dependencies resolve
- Independent operations run in parallel while dependent ones start as early as possible

### When to Apply

- Three or more async operations with partial dependencies
- Complex data fetching where some calls depend on others
- API aggregation endpoints that combine multiple data sources

### When to Skip

- Simple cases with only two sequential operations (overhead not worth it)
- All operations are truly independent (use plain `Promise.all`)
- All operations are strictly sequential (simple await chain is clearer)
- Bundle size constraints where adding a dependency isn't justified

### Expected Impact

2-10x latency reduction when operations have partial dependencies. By starting each task at the earliest possible moment rather than waiting for all dependencies to resolve, you minimize idle time and maximize parallelism.

### See Also

- `async-parallel.md` - Basic parallelization with Promise.all
- `async-defer-await.md` - Defer operations until needed

Reference: [https://github.com/shuding/better-all](https://github.com/shuding/better-all)
