---
title: Defer Await Until Needed
impact: HIGH
---

## Defer Await Until Needed

> **Review:** `await` before `if` or early return statements
> **Writing:** Async function with conditional branches that may not need fetched data
> **Technique:** Move await into the branch that actually uses the result

Blocks unused code paths, adding unnecessary latency when early-exit conditions are met.

### Fail

```typescript
// ❌ Awaits data before checking if it's needed
async function handleRequest(userId: string, skipProcessing: boolean) {
  const userData = await fetchUserData(userId)

  if (skipProcessing) {
    // Returns immediately but still waited for userData
    return { skipped: true }
  }

  // Only this branch uses userData
  return processUserData(userData)
}
```

```typescript
// ❌ Fetches permissions before checking if resource exists
async function updateResource(resourceId: string, userId: string) {
  const permissions = await fetchPermissions(userId)
  const resource = await getResource(resourceId)

  if (!resource) {
    return { error: 'Not found' }
  }

  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }

  return await updateResourceData(resource, permissions)
}
```

### Pass

```typescript
// ✅ Checks condition first, only fetches when needed
async function handleRequest(userId: string, skipProcessing: boolean) {
  if (skipProcessing) {
    // Returns immediately without waiting
    return { skipped: true }
  }

  // Fetch only when needed
  const userData = await fetchUserData(userId)
  return processUserData(userData)
}
```

```typescript
// ✅ Orders awaits to fail fast on cheap checks
async function updateResource(resourceId: string, userId: string) {
  const resource = await getResource(resourceId)

  if (!resource) {
    return { error: 'Not found' }
  }

  const permissions = await fetchPermissions(userId)

  if (!permissions.canEdit) {
    return { error: 'Forbidden' }
  }

  return await updateResourceData(resource, permissions)
}
```

### Why This Matters

- Avoids blocking code paths that don't need the fetched data
- Especially valuable when the skipped branch is frequently taken
- More impactful when the deferred operation is expensive

### When to Apply

- Functions with early return conditions before data is used
- Conditional branches where only some paths need async data
- Guard clauses that can short-circuit before fetching

### When to Skip

- All branches genuinely need the data
- The async operation is very fast (sub-millisecond)
- Moving the await would make the code significantly harder to read
- The early return is rarely taken and optimization isn't needed

### Expected Impact

Avoids blocking unused code paths, eliminating unnecessary latency when early-exit conditions are met. The benefit scales with how often the skipped branch is taken and how expensive the deferred operation is.

### See Also

- `async-parallel.md` - Parallelize independent operations
- `async-dependencies.md` - Handle partial dependencies between operations
