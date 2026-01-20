---
title: Build Index Maps for Repeated Lookups
impact: LOW-MEDIUM
---

## Build Index Maps for Repeated Lookups

> **Review:** Look for `.find()` calls inside `.map()`, `.forEach()`, or loops
> **Writing:** When joining two arrays by a shared key (id, userId, etc.)
> **Technique:** Build a Map from the lookup array first, then use `.get()` for O(1) access

Multiple `.find()` calls by the same key create O(n^2) complexity. For 1000 orders x 1000 users: 1M ops become 2K ops with a Map.

### Fail

```typescript
// ❌ O(n) lookup per iteration = O(n^2) total
function processOrders(orders: Order[], users: User[]) {
  return orders.map(order => ({
    ...order,
    user: users.find(u => u.id === order.userId)
  }))
}
```

### Pass

```typescript
// ✅ Build map once O(n), then O(1) per lookup
function processOrders(orders: Order[], users: User[]) {
  const userById = new Map(users.map(u => [u.id, u]))

  return orders.map(order => ({
    ...order,
    user: userById.get(order.userId)
  }))
}
```

### Why This Matters

- `.find()` scans the array from the start on every call
- Map lookups are O(1) regardless of array size
- Performance gap grows quadratically with data size

### When to Apply

- Joining arrays by a shared identifier
- Multiple lookups against the same array in a loop
- Processing relational data (orders/users, posts/authors, etc.)

### When to Skip

- Single lookup (not in a loop)
- Small arrays where overhead of Map creation exceeds savings
- When the lookup array changes between iterations

### Expected Impact

Transforms O(n²) nested loops into O(n) lookups by building an index Map upfront. For typical datasets, this turns 1 million operations into 2,000—a 500x improvement that grows with data size.

### See Also

- `js-set-map-lookups.md` - Use Set/Map for membership checks
- `js-combine-iterations.md` - Reduce multiple array passes
