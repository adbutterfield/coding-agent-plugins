---
title: Use Lazy State Initialization
impact: MEDIUM
---

## Use Lazy State Initialization

> **Review:** `useState(expensiveFunction())` or `useState(JSON.parse(...))` without arrow function wrapper
> **Writing:** Initializing state from localStorage, building data structures, or any non-trivial computation
> **Technique:** Pass a function to `useState`: `useState(() => expensiveFunction())`

Pass a function to `useState` for expensive initial values. Without the function form, the initializer runs on every render even though the value is only used once.

### Fail

```tsx
// ❌ buildSearchIndex() runs on EVERY render, even after initialization
function FilteredList({ items }: { items: Item[] }) {
  const [searchIndex, setSearchIndex] = useState(buildSearchIndex(items))
  const [query, setQuery] = useState('')

  // When query changes, buildSearchIndex runs again unnecessarily
  return <SearchResults index={searchIndex} query={query} />
}
```

```tsx
// ❌ JSON.parse runs on every render
function UserProfile() {
  const [settings, setSettings] = useState(
    JSON.parse(localStorage.getItem('settings') || '{}')
  )

  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

### Pass

```tsx
// ✅ buildSearchIndex() runs ONLY on initial render
function FilteredList({ items }: { items: Item[] }) {
  const [searchIndex, setSearchIndex] = useState(() => buildSearchIndex(items))
  const [query, setQuery] = useState('')

  return <SearchResults index={searchIndex} query={query} />
}
```

```tsx
// ✅ JSON.parse runs only on initial render
function UserProfile() {
  const [settings, setSettings] = useState(() => {
    const stored = localStorage.getItem('settings')
    return stored ? JSON.parse(stored) : {}
  })

  return <SettingsForm settings={settings} onChange={setSettings} />
}
```

### Why This Matters

- Without lazy initialization, expensive computations run on every render
- The initial value is only used once, but the expression is evaluated every time
- Can cause significant performance issues with complex data transformations

### When to Apply

- Computing initial values from localStorage/sessionStorage
- Building data structures (indexes, maps, sets)
- Reading from the DOM
- Heavy transformations or parsing (JSON.parse, etc.)
- Any computation that takes measurable time

### When to Skip

- Simple primitives: `useState(0)`, `useState('')`, `useState(false)`
- Direct prop references: `useState(props.initialValue)`
- Cheap literals: `useState({})`, `useState([])`
- Values that are already computed/cached

### Expected Impact

Avoids wasted computation on every render by running expensive initializers only once. For components with costly initial computations like building search indexes or parsing JSON from localStorage, this eliminates repeated work and keeps subsequent renders fast.

### See Also

- `rerender-memo.md` - Memoizing expensive computations in render
- `rerender-functional-setstate.md` - Functional updates for state that depends on previous value
