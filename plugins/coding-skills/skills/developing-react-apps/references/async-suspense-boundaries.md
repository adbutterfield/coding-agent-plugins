---
title: Strategic Suspense Boundaries
impact: HIGH
---

## Strategic Suspense Boundaries

> **Review:** `await` in async component before returning JSX with static wrapper elements
> **Writing:** Page or component with static layout and one async data section
> **Technique:** Wrap async content in Suspense, move await into child component

Blocks initial paint, adding latency for static content that could render immediately.

### Fail

```tsx
// ❌ Wrapper blocked by data fetching - entire page waits
async function Page() {
  const data = await fetchData() // Blocks entire page

  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <DataDisplay data={data} />
      </div>
      <div>Footer</div>
    </div>
  )
}
```

### Pass

```tsx
// ✅ Wrapper shows immediately, data streams in
function Page() {
  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <div>
        <Suspense fallback={<Skeleton />}>
          <DataDisplay />
        </Suspense>
      </div>
      <div>Footer</div>
    </div>
  )
}

async function DataDisplay() {
  const data = await fetchData() // Only blocks this component
  return <div>{data.content}</div>
}
```

```tsx
// ✅ Alternative: share promise across components
function Page() {
  // Start fetch immediately, but don't await
  const dataPromise = fetchData()

  return (
    <div>
      <div>Sidebar</div>
      <div>Header</div>
      <Suspense fallback={<Skeleton />}>
        <DataDisplay dataPromise={dataPromise} />
        <DataSummary dataPromise={dataPromise} />
      </Suspense>
      <div>Footer</div>
    </div>
  )
}

function DataDisplay({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // Unwraps the promise
  return <div>{data.content}</div>
}

function DataSummary({ dataPromise }: { dataPromise: Promise<Data> }) {
  const data = use(dataPromise) // Reuses the same promise
  return <div>{data.summary}</div>
}
```

### Why This Matters

- Static layout (Sidebar, Header, Footer) renders immediately
- Only the async section shows a loading state
- Multiple components can share the same promise for a single fetch

### When to Apply

- Pages with static wrapper elements and async content sections
- Layouts where most content is static but some parts need data
- Multiple components that consume the same async data

### When to Skip

- Critical data needed for layout decisions (affects positioning/dimensions)
- SEO-critical content above the fold that must be in initial HTML
- Small, fast queries where suspense overhead isn't worth it
- When layout shift would create poor UX (content jump after load)
- Data fetch is sub-50ms and blocking is imperceptible

### Expected Impact

Faster initial paint by streaming content progressively. Users see static layout elements immediately while async content loads in place, improving perceived performance and reducing time to first meaningful paint.

### See Also

- `async-parallel.md` - Parallelize multiple data fetches within suspense boundaries
- `async-defer-await.md` - Defer operations until needed
