---
title: Use SWR for Automatic Deduplication
impact: MEDIUM-HIGH
---

## Use SWR for Automatic Deduplication

> **Review:** `useEffect` with `fetch` inside components, especially with `useState` + `setX` pattern
> **Writing:** When adding data fetching to a component that may be rendered multiple times
> **Technique:** Replace useEffect/fetch with useSWR for automatic request deduplication and caching

SWR enables request deduplication, caching, and revalidation across component instances. Multiple components using the same key share one request.

### Fail

```tsx
// ❌ No deduplication - each instance fetches independently
function UserList() {
  const [users, setUsers] = useState([])
  useEffect(() => {
    fetch('/api/users')
      .then(r => r.json())
      .then(setUsers)
  }, [])
}
```

### Pass

```tsx
// ✅ Multiple instances share one request with automatic caching
import useSWR from 'swr'

function UserList() {
  const { data: users } = useSWR('/api/users', fetcher)
}
```

```tsx
// ✅ For immutable/static data that never changes
import { useImmutableSWR } from '@/lib/swr'

function StaticContent() {
  const { data } = useImmutableSWR('/api/config', fetcher)
}
```

```tsx
// ✅ For mutations with optimistic updates
import { useSWRMutation } from 'swr/mutation'

function UpdateButton() {
  const { trigger } = useSWRMutation('/api/user', updateUser)
  return <button onClick={() => trigger()}>Update</button>
}
```

### Why This Matters

- Multiple component instances making the same request wastes bandwidth and server resources
- SWR provides built-in caching, revalidation, and error handling
- Automatic deduplication means N components = 1 request instead of N requests

### When to Apply

- Any component that fetches data and might be rendered multiple times
- Lists where each item fetches related data
- Shared data needed across multiple parts of the UI

### When to Skip

- One-time fetches in isolated components that are never duplicated
- Server components where data fetching happens at build/request time
- When you need fine-grained control over caching that SWR doesn't support

### Expected Impact

Automatic deduplication prevents redundant network requests when multiple components request the same data. This reduces server load and bandwidth usage proportionally to the number of duplicate component instances, turning N requests into 1.

### See Also

- `client-event-listeners.md` - Similar deduplication pattern for global event listeners
- [SWR Documentation](https://swr.vercel.app)
