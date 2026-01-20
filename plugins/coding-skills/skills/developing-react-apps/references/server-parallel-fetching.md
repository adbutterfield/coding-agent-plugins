---
title: Parallel Data Fetching with Component Composition
impact: CRITICAL
---

## Parallel Data Fetching with Component Composition

> **Review:** `async function Page()` with `await` before rendering child async components
> **Writing:** When creating pages with multiple independent data requirements
> **Technique:** Move data fetching into separate async components that render as siblings

React Server Components execute sequentially within a tree. When a parent component awaits data before rendering children, those children cannot start their own data fetching until the parent completes. Restructure with composition to parallelize data fetching and eliminate server-side waterfalls.

### Fail

```tsx
// ❌ Sidebar waits for Page's fetch to complete before starting its own fetch
export default async function Page() {
  const header = await fetchHeader()
  return (
    <div>
      <div>{header}</div>
      <Sidebar />
    </div>
  )
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}
```

### Pass

```tsx
// ✅ Both components fetch simultaneously - no waterfall
async function Header() {
  const data = await fetchHeader()
  return <div>{data}</div>
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}

export default function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
    </div>
  )
}
```

**Alternative with children prop:**

```tsx
// ✅ Layout pattern with parallel fetching
async function Header() {
  const data = await fetchHeader()
  return <div>{data}</div>
}

async function Sidebar() {
  const items = await fetchSidebarItems()
  return <nav>{items.map(renderItem)}</nav>
}

function Layout({ children }: { children: ReactNode }) {
  return (
    <div>
      <Header />
      {children}
    </div>
  )
}

export default function Page() {
  return (
    <Layout>
      <Sidebar />
    </Layout>
  )
}
```

### Why This Matters

- Sequential fetching creates waterfall delays that compound with each nested await
- Parallel fetching can dramatically reduce total page load time
- This is one of the most impactful performance optimizations for RSC applications

### When to Apply

- Pages with multiple independent data requirements
- Layouts with header, sidebar, and content that fetch different data
- Any async Server Component tree where children don't depend on parent data

### When to Skip

- When child components genuinely depend on parent data (true data dependencies)
- Single data requirement pages with no parallelization opportunity
- When using Suspense boundaries that already handle loading states appropriately

### Expected Impact

Eliminates server-side waterfalls, reducing page load time proportionally to the number of sequential fetches removed. For pages with multiple independent data requirements, this can cut total server response time from the sum of all fetch durations to the duration of the slowest single fetch.

### See Also

- `async-parallel.md` - Promise.all for parallel async operations
- `server-cache-react.md` - Per-request deduplication with React.cache()
- `async-suspense-boundaries.md` - Suspense boundaries for loading states
