---
title: Use Loop for Min/Max Instead of Sort
impact: LOW
---

## Use Loop for Min/Max Instead of Sort

> **Review:** `.sort()` followed by `[0]` or `[length - 1]` to get min/max
> **Writing:** When finding the smallest, largest, oldest, or newest item in an array
> **Technique:** Use a single-pass loop to track the extremum

O(n) instead of O(n log n). Finding min/max only requires one pass through the array; sorting is wasteful.

### Fail

```typescript
// ❌ O(n log n) - sorts entire array just to find the latest
interface Project {
  id: string
  name: string
  updatedAt: number
}

function getLatestProject(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => b.updatedAt - a.updatedAt)
  return sorted[0]
}

// ❌ O(n log n) - sorts to find both oldest and newest
function getOldestAndNewest(projects: Project[]) {
  const sorted = [...projects].sort((a, b) => a.updatedAt - b.updatedAt)
  return { oldest: sorted[0], newest: sorted[sorted.length - 1] }
}
```

### Pass

```typescript
// ✅ O(n) - single loop to find latest
function getLatestProject(projects: Project[]) {
  if (projects.length === 0) return null

  let latest = projects[0]

  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt > latest.updatedAt) {
      latest = projects[i]
    }
  }

  return latest
}

// ✅ O(n) - single loop finds both oldest and newest
function getOldestAndNewest(projects: Project[]) {
  if (projects.length === 0) return { oldest: null, newest: null }

  let oldest = projects[0]
  let newest = projects[0]

  for (let i = 1; i < projects.length; i++) {
    if (projects[i].updatedAt < oldest.updatedAt) oldest = projects[i]
    if (projects[i].updatedAt > newest.updatedAt) newest = projects[i]
  }

  return { oldest, newest }
}
```

### Why This Matters

- Single pass through the array, no copying, no sorting overhead
- Memory efficient: no intermediate sorted array created
- Clearer intent: code explicitly shows you want min/max, not a sorted list

### When to Apply

- Finding the most recent, oldest, highest, lowest item
- Dashboard "top item" displays
- Any scenario where you only need 1-2 extreme values

### When to Skip

- When you need the top N items (N > 2) - sorting may be appropriate
- When you need the sorted array for display purposes anyway
- Very small arrays (< 10 items) where the difference is negligible

### Alternative for Simple Numbers

```typescript
// Works for small arrays of primitives
const numbers = [5, 2, 8, 1, 9]
const min = Math.min(...numbers)
const max = Math.max(...numbers)
```

Note: This can throw for very large arrays due to spread operator limits (~124k in Chrome, ~638k in Safari). Use the loop approach for reliability with unknown array sizes.

### Expected Impact

Reduces complexity from O(n log n) for sorting to O(n) for a single pass, which becomes significant for large arrays. For an array of 10,000 items, this means roughly 10,000 comparisons instead of 130,000.

### See Also

- `js-early-exit.md` - Early return patterns for loop optimization
