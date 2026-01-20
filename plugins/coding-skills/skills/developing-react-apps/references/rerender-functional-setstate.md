---
title: Use Functional setState Updates
impact: MEDIUM
---

## Use Functional setState Updates

> **Review:** `setState` referencing state variable in dependency array, or `setState` with missing state dependency
> **Writing:** Updating state based on current state value, especially in callbacks
> **Technique:** Use `setState(prev => ...)` instead of `setState(value)`

Functional updates prevent stale closures and eliminate unnecessary callback recreations. When state is referenced directly in `setState`, callbacks must include it as a dependency, causing recreations on every state change.

### Fail

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)

  // ❌ Callback must depend on items, recreated on every items change
  const addItems = useCallback((newItems: Item[]) => {
    setItems([...items, ...newItems])
  }, [items])

  // ❌ Missing items dependency - will use stale items!
  const removeItem = useCallback((id: string) => {
    setItems(items.filter(item => item.id !== id))
  }, [])

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}
```

### Pass

```tsx
function TodoList() {
  const [items, setItems] = useState(initialItems)

  // ✅ Stable callback, never recreated
  const addItems = useCallback((newItems: Item[]) => {
    setItems(curr => [...curr, ...newItems])
  }, [])

  // ✅ Always uses latest state, no stale closure risk
  const removeItem = useCallback((id: string) => {
    setItems(curr => curr.filter(item => item.id !== id))
  }, [])

  return <ItemsEditor items={items} onAdd={addItems} onRemove={removeItem} />
}
```

### Why This Matters

- Stable callback references prevent child component re-renders
- No stale closures—always operates on the latest state value
- Fewer dependencies simplify dependency arrays and reduce bugs

### When to Apply

- Any `setState` that depends on the current state value
- Inside `useCallback`/`useMemo` when state is needed
- Event handlers that reference state
- Async operations that update state

### When to Skip

- Setting state to a static value: `setCount(0)`
- Setting state from props/arguments only: `setName(newName)`
- State doesn't depend on previous value

### Expected Impact

Prevents stale closure bugs and eliminates unnecessary callback recreations, leading to more stable component trees. Callbacks with empty dependency arrays maintain referential equality across renders, allowing memoized child components to skip re-rendering when passed as props.

### See Also

- `rerender-memo.md` - Memoization patterns
- `rerender-callback-ref.md` - Stable callback references

**Note:** If your project has [React Compiler](https://react.dev/learn/react-compiler) enabled, the compiler can automatically optimize some cases, but functional updates are still recommended for correctness and to prevent stale closure bugs.
