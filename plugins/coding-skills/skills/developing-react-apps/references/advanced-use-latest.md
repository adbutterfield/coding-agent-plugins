---
title: useLatest for Stable Callback Refs
impact: LOW
---

## useLatest for Stable Callback Refs

> **Review:** `useEffect` with callback prop in dependency array causing unwanted re-runs
> **Writing:** Need to access a prop or state value inside an effect without it triggering re-runs
> **Technique:** Store the value in a ref via `useLatest` hook, access via `.current` in the effect

Access latest values in callbacks without adding them to dependency arrays. Prevents effect re-runs while avoiding stale closures.

### Implementation

```typescript
function useLatest<T>(value: T) {
  const ref = useRef(value)
  useLayoutEffect(() => {
    ref.current = value
  }, [value])
  return ref
}
```

### Fail

```tsx
// ❌ Effect re-runs on every onSearch callback change, resetting the debounce timer
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')

  useEffect(() => {
    const timeout = setTimeout(() => onSearch(query), 300)
    return () => clearTimeout(timeout)
  }, [query, onSearch])
}
```

### Pass

```tsx
// ✅ Stable effect that only re-runs when query changes, always calls fresh callback
function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState('')
  const onSearchRef = useLatest(onSearch)

  useEffect(() => {
    const timeout = setTimeout(() => onSearchRef.current(query), 300)
    return () => clearTimeout(timeout)
  }, [query])
}
```

### Why This Matters

- Callback props from parent components typically have unstable identity
- Including unstable callbacks in dependency arrays causes unnecessary effect re-runs
- Stale closures are a common source of bugs; refs ensure you always call the latest version

### When to Apply

- Debounced or throttled callbacks that receive handler props
- Effects with timers or intervals that call external callbacks
- Event subscription effects where the handler is passed as a prop
- Any effect where you need to read a value but not react to its changes

### When to Skip

- When the callback is already memoized with stable dependencies
- When you actually want the effect to re-run when the callback changes
- React 19+ projects where `useEffectEvent` is available (use that instead)
- Simple effects where the extra indirection adds unnecessary complexity

### Expected Impact

Prevents effect re-runs when callback content changes but effect dependencies should remain stable. This keeps timers, debounce logic, and other time-sensitive effects running smoothly without being interrupted by unrelated re-renders, while ensuring the latest callback is always invoked when needed.

### See Also

- `advanced-event-handler-refs.md` - Specific pattern for event listener subscriptions
- `rerender-dependencies.md` - Narrowing dependencies to minimize effect re-runs
