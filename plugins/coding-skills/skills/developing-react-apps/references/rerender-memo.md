---
title: Extract to Memoized Components
impact: MEDIUM
---

## Extract to Memoized Components

> **Review:** `useMemo` computing JSX before an early return, or expensive computation in components with loading/conditional states
> **Writing:** Building a component with expensive computation that also has loading or conditional rendering states
> **Technique:** Extract expensive work into a separate `memo()` component rendered after early returns

Extracting expensive work into memoized child components enables early returns before computation. When `useMemo` is in the parent, it runs regardless of whether the result is used.

### Fail

```tsx
// ❌ Computes avatar even when loading (useMemo runs before early return)
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => {
    const id = computeAvatarId(user)
    return <Avatar id={id} />
  }, [user])

  if (loading) return <Skeleton />
  return <div>{avatar}</div>
}
```

### Pass

```tsx
// ✅ Computation skipped entirely when parent shows loading state
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const id = useMemo(() => computeAvatarId(user), [user])
  return <Avatar id={id} />
})

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />
  return (
    <div>
      <UserAvatar user={user} />
    </div>
  )
}
```

### Why This Matters

- Child components don't render at all when parent returns early
- `memo()` provides props comparison to skip re-renders when props haven't changed
- Separates concerns: parent handles loading state, child handles expensive computation

### When to Apply

- Components with early returns (loading, error, empty states) that also have expensive computations
- `useMemo` that computes JSX elements
- Expensive operations that could be isolated into a self-contained component

### When to Skip

- The computation is cheap and the overhead of an extra component isn't worth it
- The computed value is needed before all early returns
- React Compiler is enabled (it automatically optimizes re-renders)

### Expected Impact

Enables React to bail out early when props haven't changed, skipping expensive subtree renders. Memoized components with stable props can avoid re-rendering entirely, leading to measurable performance gains in components with complex children or expensive computations.

### See Also

- `rerender-functional-setstate.md` - Stable callback references for memoized components
- `rerender-dependencies.md` - Narrowing dependencies to minimize re-computation

**Note:** If your project has [React Compiler](https://react.dev/learn/react-compiler) enabled, manual memoization with `memo()` and `useMemo()` is not necessary. The compiler automatically optimizes re-renders.
