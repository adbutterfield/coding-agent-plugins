---
title: Early Length Check for Array Comparisons
impact: MEDIUM-HIGH
---

## Early Length Check for Array Comparisons

> **Review:** Array comparisons using `.sort()`, `.join()`, or deep equality without length check
> **Writing:** When comparing two arrays for equality or detecting changes
> **Technique:** Check `array1.length !== array2.length` before expensive operations

Avoids expensive operations when lengths differ. In hot paths (event handlers, render loops), this O(1) check can skip O(n log n) sorting entirely.

### Fail

```typescript
// ❌ Always sorts and joins, even when lengths differ
function hasChanges(current: string[], original: string[]) {
  return current.sort().join() !== original.sort().join()
}
```

### Pass

```typescript
// ✅ O(1) length check skips expensive operations when arrays differ in size
function hasChanges(current: string[], original: string[]) {
  if (current.length !== original.length) {
    return true
  }
  const currentSorted = current.toSorted()
  const originalSorted = original.toSorted()
  for (let i = 0; i < currentSorted.length; i++) {
    if (currentSorted[i] !== originalSorted[i]) {
      return true
    }
  }
  return false
}
```

### Why This Matters

- Avoids O(n log n) sorting when a simple O(1) check can determine inequality
- Prevents memory allocation for intermediate strings (join) or arrays (spread)
- Returns early when a difference is found during element-by-element comparison

### When to Apply

- Comparing arrays in React state change detection
- Equality checks in useMemo/useEffect dependency logic
- Any array comparison before sorting, serializing, or deep equality checks

### When to Skip

- When you need the sorted result regardless of equality (not just checking)
- When arrays are guaranteed to always have the same length by design
- Very small arrays (< 5 elements) where the overhead is negligible

### Expected Impact

Avoids expensive deep comparisons when array lengths differ, providing an O(1) early exit. This optimization can completely skip O(n log n) sorting operations when arrays have different lengths, which is often the case in real-world change detection scenarios.

### See Also

- `js-early-exit.md` - General early return pattern
- `js-tosorted-immutable.md` - Use toSorted() to avoid mutating original arrays
