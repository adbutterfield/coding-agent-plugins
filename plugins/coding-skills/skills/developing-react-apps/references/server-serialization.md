---
title: Minimize Serialization at RSC Boundaries
impact: HIGH
---

## Minimize Serialization at RSC Boundaries

> **Review:** Passing entire objects to 'use client' components when only a few fields are used
> **Writing:** When passing data from Server Components to Client Components
> **Technique:** Extract and pass only the specific fields needed by the client component

The React Server/Client boundary serializes all object properties into strings and embeds them in the HTML response. This serialized data directly impacts page weight and load time, so size matters a lot. Only pass fields that the client actually uses.

### Fail

```tsx
// ❌ Serializes all 50 fields when client only uses 1
async function Page() {
  const user = await fetchUser()  // 50 fields
  return <Profile user={user} />
}

'use client'
function Profile({ user }: { user: User }) {
  return <div>{user.name}</div>  // uses 1 field
}
```

### Pass

```tsx
// ✅ Serializes only the 1 field actually needed
async function Page() {
  const user = await fetchUser()
  return <Profile name={user.name} />
}

'use client'
function Profile({ name }: { name: string }) {
  return <div>{name}</div>
}
```

### Why This Matters

- Every byte crossing the RSC boundary adds to page weight
- Large objects with unused fields waste bandwidth and slow initial load
- Smaller props mean faster hydration and better Core Web Vitals

### When to Apply

- Any Server Component passing data to a Client Component
- Components receiving database models or API responses
- When client components only need a subset of available data

### When to Skip

- When the client component genuinely needs most/all fields from the object
- For small objects where the overhead of extraction exceeds the benefit
- When the data structure is already minimal

### Expected Impact

Reduces data transfer size between server and client components, improving Time to First Byte (TTFB) and initial page load performance. For components receiving large objects where only a few fields are used, this can reduce serialized payload size by 50-90%, directly improving Core Web Vitals.

### See Also

- `server-parallel-fetching.md` - Parallel data fetching with component composition
- `bundle-dynamic-imports.md` - Code splitting for client components
