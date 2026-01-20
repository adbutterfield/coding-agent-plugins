---
title: Combine Multiple Array Iterations
impact: LOW-MEDIUM
---

## Combine Multiple Array Iterations

> **Review:** Look for multiple `.filter()` or `.map()` calls on the same array
> **Writing:** When extracting multiple subsets from a single array
> **Technique:** Use a single `for...of` loop with multiple result arrays

Multiple `.filter()` or `.map()` calls iterate the array multiple times. Combine into one loop to reduce iterations.

### Fail

```typescript
// ❌ 3 iterations over the same array
const admins = users.filter(u => u.isAdmin)
const testers = users.filter(u => u.isTester)
const inactive = users.filter(u => !u.isActive)
```

### Pass

```typescript
// ✅ 1 iteration, classify each item once
const admins: User[] = []
const testers: User[] = []
const inactive: User[] = []

for (const user of users) {
  if (user.isAdmin) admins.push(user)
  if (user.isTester) testers.push(user)
  if (!user.isActive) inactive.push(user)
}
```

### Why This Matters

- Each `.filter()` or `.map()` creates a full pass over the array
- Memory allocation for intermediate arrays adds GC pressure
- Single loop with pushes is more cache-friendly

### When to Apply

- Categorizing items into multiple groups
- Extracting multiple derived values from the same source
- Processing large arrays where iteration cost matters

### When to Skip

- Single filter/map operation (no benefit)
- Small arrays where readability outweighs performance
- When chained operations are clearer and performance is acceptable

### Expected Impact

Reduces total iterations by combining multiple array passes into one. If you have 3 filter operations on a 1,000-item array, this cuts 3,000 iterations down to 1,000—proportional savings that scale with array size and number of operations.

### See Also

- `js-index-maps.md` - Build Maps for O(1) lookups
- `js-cache-property-access.md` - Cache property lookups in loops
- `js-early-exit.md` - Exit loops early when possible
