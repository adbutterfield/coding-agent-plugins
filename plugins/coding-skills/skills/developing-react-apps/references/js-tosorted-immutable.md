---
title: Use toSorted() Instead of sort() for Immutability
impact: MEDIUM-HIGH
---

## Use toSorted() Instead of sort() for Immutability

> **Review:** `.sort()` on arrays that are props, state, or derived from them
> **Writing:** When sorting arrays in React components
> **Technique:** Use `.toSorted()` to create a new sorted array without mutating the original

Prevents mutation bugs in React state and props. `.sort()` mutates in place, breaking React's immutability model and causing subtle bugs.

### Fail

```typescript
// ❌ Mutates the users prop array!
function UserList({ users }: { users: User[] }) {
  const sorted = useMemo(
    () => users.sort((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

### Pass

```typescript
// ✅ Creates new sorted array, original unchanged
function UserList({ users }: { users: User[] }) {
  const sorted = useMemo(
    () => users.toSorted((a, b) => a.name.localeCompare(b.name)),
    [users]
  )
  return <div>{sorted.map(renderUser)}</div>
}
```

### Why This Matters

- Props and state mutations break React's immutability model
- Causes stale closure bugs when mutating arrays inside callbacks or effects
- React may not detect changes, leading to missed re-renders

### When to Apply

- Sorting props or state arrays in React components
- Any sorting operation where the original array should remain unchanged
- Inside useMemo, useCallback, or event handlers

### When to Skip

- Sorting a locally-created array that will never be referenced again
- Non-React code where mutation is intentional and documented
- Performance-critical paths in older environments (see fallback below)

### Browser Support

`.toSorted()` is available in modern browsers (Chrome 110+, Safari 16+, Firefox 115+, Node.js 20+). For older environments:

```typescript
// Fallback: spread then sort
const sorted = [...items].sort((a, b) => a.value - b.value)
```

### Other Immutable Array Methods

- `.toSorted()` - immutable sort
- `.toReversed()` - immutable reverse
- `.toSpliced()` - immutable splice
- `.with(index, value)` - immutable element replacement

### Expected Impact

Prevents mutation bugs in React state by using non-mutating sort, avoiding subtle re-render issues. This eliminates an entire category of bugs where React fails to detect changes because the original array reference was mutated in place.

### See Also

- `js-length-check-first.md` - Check length before expensive array operations
