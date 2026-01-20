---
title: Cache Property Access in Loops
impact: LOW-MEDIUM
---

## Cache Property Access in Loops

> **Review:** Look for deep property chains like `obj.config.settings.value` inside loops
> **Writing:** When accessing the same nested property repeatedly in a hot path
> **Technique:** Extract the property to a local variable before the loop

Cache object property lookups in hot paths. Each `.` traversal is a lookup operation that adds up across iterations.

### Fail

```typescript
// ❌ 3 property lookups x N iterations
for (let i = 0; i < arr.length; i++) {
  process(obj.config.settings.value)
}
```

### Pass

```typescript
// ✅ 1 lookup total, cached before the loop
const value = obj.config.settings.value
const len = arr.length
for (let i = 0; i < len; i++) {
  process(value)
}
```

### Why This Matters

- Each property access requires engine lookup through the prototype chain
- `arr.length` is re-evaluated on every iteration in some engines
- Local variables are faster to access than object properties

### When to Apply

- Loops with many iterations (hundreds or more)
- Deep property chains accessed in loops
- Performance-critical rendering or data processing code

### When to Skip

- Short loops (< 10 iterations) where overhead is negligible
- When the property value might change during iteration
- Modern engines optimize simple cases; profile before optimizing

### Expected Impact

Reduces repeated property lookups in hot loops by caching values in local variables. This improves iteration speed in performance-critical code by eliminating redundant prototype chain traversals.

### See Also

- `js-cache-function-results.md` - Cache expensive function return values
- `js-combine-iterations.md` - Reduce multiple array passes
