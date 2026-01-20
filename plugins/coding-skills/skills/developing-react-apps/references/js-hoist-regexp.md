---
title: Hoist RegExp Creation
impact: LOW-MEDIUM
---

## Hoist RegExp Creation

> **Review:** `new RegExp()` inside component body or render function
> **Writing:** When using regular expressions in React components
> **Technique:** Hoist static patterns to module scope; memoize dynamic patterns with `useMemo()`

Avoids recreation of RegExp objects on every render. RegExp compilation has overhead that adds up in frequently-rendered components.

### Fail

```tsx
// ❌ Creates new RegExp on every render
function Highlighter({ text, query }: Props) {
  const regex = new RegExp(`(${query})`, 'gi')
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

### Pass

```tsx
// ✅ Static patterns hoisted to module scope
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

// ✅ Dynamic patterns memoized
function Highlighter({ text, query }: Props) {
  const regex = useMemo(
    () => new RegExp(`(${escapeRegex(query)})`, 'gi'),
    [query]
  )
  const parts = text.split(regex)
  return <>{parts.map((part, i) => ...)}</>
}
```

### Why This Matters

- RegExp compilation is not free; hoisting eliminates repeated work
- Memoization ensures dynamic patterns only recompile when inputs change
- Escaping user input prevents regex injection vulnerabilities

### When to Apply

- Any RegExp used inside a React component
- Validation patterns (email, phone, URL, etc.)
- Search/highlight functionality with user-provided patterns

### When to Skip

- One-time utility functions outside React render cycle
- Server-side code that runs once per request
- Patterns that genuinely change on every call (rare)

### Gotchas

Global regex (`/g`) has mutable `lastIndex` state:

```typescript
const regex = /foo/g
regex.test('foo')  // true, lastIndex = 3
regex.test('foo')  // false, lastIndex = 0
```

Reset `lastIndex` or use a fresh regex for repeated tests on different strings.

### Expected Impact

Avoids recreating RegExp objects on every call, eliminating repeated regex compilation overhead. In frequently-rendered components, this can prevent hundreds or thousands of unnecessary regex compilations per second.

### See Also

- `react-memo-expensive-calculations.md` - General memoization patterns
