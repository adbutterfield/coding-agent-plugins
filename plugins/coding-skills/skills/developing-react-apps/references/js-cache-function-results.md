---
title: Cache Repeated Function Calls
impact: MEDIUM
---

## Cache Repeated Function Calls

> **Review:** Look for pure function calls inside `.map()`, `.forEach()`, or render methods
> **Writing:** When the same function is called multiple times with identical inputs
> **Technique:** Use a module-level Map to cache results by input

Use a module-level Map to cache function results when the same function is called repeatedly with the same inputs during render. This avoids redundant computation.

### Fail

```typescript
// ❌ slugify() called 100+ times for same project names
function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        const slug = slugify(project.name)

        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

### Pass

```typescript
// ✅ Computed only once per unique project name
const slugifyCache = new Map<string, string>()

function cachedSlugify(text: string): string {
  if (slugifyCache.has(text)) {
    return slugifyCache.get(text)!
  }
  const result = slugify(text)
  slugifyCache.set(text, result)
  return result
}

function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div>
      {projects.map(project => {
        const slug = cachedSlugify(project.name)

        return <ProjectCard key={project.id} slug={slug} />
      })}
    </div>
  )
}
```

### Simpler Pattern for Single-Value Functions

```typescript
// ✅ Simple cache for functions with no arguments
let isLoggedInCache: boolean | null = null

function isLoggedIn(): boolean {
  if (isLoggedInCache !== null) {
    return isLoggedInCache
  }

  isLoggedInCache = document.cookie.includes('auth=')
  return isLoggedInCache
}

function onAuthChange() {
  isLoggedInCache = null  // Clear cache when auth changes
}
```

### Why This Matters

- Module-level Map works everywhere: utilities, event handlers, not just React components
- Avoids expensive recomputation across re-renders
- Reference: [How we made the Vercel Dashboard twice as fast](https://vercel.com/blog/how-we-made-the-vercel-dashboard-twice-as-fast)

### When to Apply

- Pure functions called repeatedly with same inputs
- Expensive transformations (slugify, format, parse, serialize)
- Functions called during render that don't depend on component state

### When to Skip

- Functions with side effects
- When inputs change frequently (cache provides no benefit)
- Memory-constrained environments (cache grows unbounded)
- When results depend on external mutable state

### Expected Impact

Avoids redundant expensive computations by caching function results in a module-level Map. This trades a small amount of memory for significant CPU time savings when the same inputs are processed repeatedly.

### See Also

- `js-cache-storage.md` - Cache localStorage/sessionStorage reads
- `js-cache-property-access.md` - Cache property lookups in loops
