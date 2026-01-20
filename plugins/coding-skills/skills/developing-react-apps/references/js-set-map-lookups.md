---
title: Use Set/Map for O(1) Lookups
impact: LOW-MEDIUM
---

## Use Set/Map for O(1) Lookups

> **Review:** `.includes()` or `.find()` on arrays inside loops or filter operations
> **Writing:** When checking membership repeatedly against a fixed list
> **Technique:** Convert the lookup array to a Set or Map before the loop

O(n) to O(1) per lookup. When filtering or checking membership multiple times, Set/Map provides constant-time access instead of linear scans.

### Fail

```typescript
// ❌ O(n) per check - scans allowedIds for every item
const allowedIds = ['a', 'b', 'c', ...]
items.filter(item => allowedIds.includes(item.id))
```

### Pass

```typescript
// ✅ O(1) per check - hash-based lookup
const allowedIds = new Set(['a', 'b', 'c', ...])
items.filter(item => allowedIds.has(item.id))
```

### Why This Matters

- `Array.includes()` is O(n); `Set.has()` is O(1)
- For filtering N items against M allowed values: O(N*M) becomes O(N+M)
- Particularly impactful when the lookup list is large or the operation runs frequently

### When to Apply

- Filtering arrays against a whitelist/blacklist
- Checking permissions against a list of allowed roles
- Deduplication or intersection operations
- Any repeated membership checks in hot paths

### When to Skip

- Lookup list has fewer than ~10 items (overhead of Set creation may not pay off)
- Single lookup (no benefit from pre-building the Set)
- When you need array methods like `map`, `filter`, or index-based access on the lookup list itself

### Map for Key-Value Lookups

```typescript
// ❌ O(n) lookup per item
const users = [{ id: 'a', name: 'Alice' }, { id: 'b', name: 'Bob' }]
items.map(item => users.find(u => u.id === item.userId)?.name)

// ✅ O(1) lookup per item
const userMap = new Map(users.map(u => [u.id, u]))
items.map(item => userMap.get(item.userId)?.name)
```

### Expected Impact

Transforms O(n) array searches into O(1) Set/Map lookups, dramatically improving performance for repeated membership checks. When filtering N items against M allowed values, complexity drops from O(N*M) to O(N+M).

### See Also

- `js-early-exit.md` - Exit loops early when possible
