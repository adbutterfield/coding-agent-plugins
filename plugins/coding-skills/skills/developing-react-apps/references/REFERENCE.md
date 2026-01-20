# React Performance Rules Index

This directory contains 47 performance optimization rules for React and Next.js applications. Rules are organized by category and prioritized by impact.

## How to Use This Index

**For Code Review:**
1. Grep the Keywords column for patterns in the code (e.g., `await`, `useCallback`, `localStorage`)
2. Scan the Impact column for issues matching reported symptoms (e.g., "slow", "flicker", "re-renders")
3. Read the matched rule's **Review** line for the specific anti-pattern
4. Check **When to Skip** before flagging

**For Writing New Code:**
1. Browse by category when you know the problem domain (data fetching, re-renders, bundle size)
2. Search Keywords for the API you're using (`useState`, `useEffect`, `Promise.all`)
3. Read the **Writing** line to confirm the rule applies

**For Debugging Performance Issues:**
1. Match symptoms in the Impact column (jank, latency, memory, TTI)
2. Search Keywords for tool outputs (Lighthouse, Profiler, DevTools)
3. Follow **See Also** links in rules to find related optimizations

---

## 1. Eliminating Waterfalls (CRITICAL)

Waterfalls are the #1 performance killer. Each sequential await adds full network latency.

| Rule | Description | Keywords | Impact |
|------|-------------|----------|--------|
| [async-defer-await.md](async-defer-await.md) | Move await into branches where actually used | `await`, `if`, conditional, branch, defer, early return | Avoids blocking unused code paths |
| [async-parallel.md](async-parallel.md) | Use Promise.all() for independent operations | `await`, `Promise.all`, sequential, parallel, waterfall, slow, latency | 2-10× latency reduction |
| [async-dependencies.md](async-dependencies.md) | Use better-all for partial dependencies | `await`, `Promise.all`, dependencies, partial, better-all, chained | 2-10× latency reduction |
| [async-api-routes.md](async-api-routes.md) | Start promises early, await late in API routes | `await`, API route, handler, early start, TTFB | 2-10× latency reduction |
| [async-suspense-boundaries.md](async-suspense-boundaries.md) | Use Suspense to stream content | `Suspense`, streaming, boundary, loading, LCP, slow initial load | Faster initial paint |

---

## 2. Bundle Size Optimization (CRITICAL)

Reducing initial bundle size improves Time to Interactive and Largest Contentful Paint.

| Rule | Description | Keywords | Impact |
|------|-------------|----------|--------|
| [bundle-barrel-imports.md](bundle-barrel-imports.md) | Import directly, avoid barrel files | `import`, barrel, index, tree-shaking, bundle analyzer, slow build | 200-800ms import cost |
| [bundle-dynamic-imports.md](bundle-dynamic-imports.md) | Use next/dynamic for heavy components | `dynamic`, `next/dynamic`, lazy, code-splitting, TTI, LCP, Lighthouse | Directly affects TTI/LCP |
| [bundle-defer-third-party.md](bundle-defer-third-party.md) | Load analytics/logging after hydration | analytics, third-party, `useEffect`, hydration, Sentry, Intercom | Loads after hydration |
| [bundle-conditional.md](bundle-conditional.md) | Load modules only when feature is activated | `import()`, conditional, feature flag, on-demand, large JSON | Loads only when needed |
| [bundle-preload.md](bundle-preload.md) | Preload on hover/focus for perceived speed | preload, hover, focus, `onMouseEnter`, perceived latency | Reduces perceived latency |

---

## 3. Server-Side Performance (HIGH)

Optimizing server-side rendering and data fetching eliminates server-side waterfalls and reduces response times.

| Rule | Description | Keywords | Impact |
|------|-------------|----------|--------|
| [server-cache-react.md](server-cache-react.md) | Use React.cache() for per-request deduplication | `cache`, `React.cache`, deduplication, request, RSC, duplicate fetch | Deduplicates within request |
| [server-cache-lru.md](server-cache-lru.md) | Use LRU cache for cross-request caching | LRU, cache, cross-request, `lru-cache`, database, TTFB | Caches across requests |
| [server-serialization.md](server-serialization.md) | Minimize data passed to client components | serialization, props, `use client`, data size, large object | Reduces data transfer size |
| [server-parallel-fetching.md](server-parallel-fetching.md) | Restructure components to parallelize fetches | parallel, server component, fetch, restructure, RSC waterfall | Eliminates server waterfalls |
| [server-after-nonblocking.md](server-after-nonblocking.md) | Use after() for non-blocking operations | `after`, non-blocking, logging, analytics, Next.js 15 | Faster response times |

---

## 4. Client-Side Data Fetching (MEDIUM-HIGH)

Automatic deduplication and efficient data fetching patterns reduce redundant network requests.

| Rule | Description | Keywords | Impact |
|------|-------------|----------|--------|
| [client-swr-dedup.md](client-swr-dedup.md) | Use SWR for automatic request deduplication | SWR, `useSWR`, deduplication, fetch, caching, duplicate requests | Automatic deduplication |
| [client-event-listeners.md](client-event-listeners.md) | Deduplicate global event listeners | `addEventListener`, global, window, deduplication, memory leak | Single listener for N components |
| [client-localstorage-schema.md](client-localstorage-schema.md) | Schema versioning for localStorage | `localStorage`, schema, version, migration, corrupt data | Prevents schema conflicts |
| [client-passive-event-listeners.md](client-passive-event-listeners.md) | Use passive event listeners for scroll/touch | passive, scroll, touch, wheel, `addEventListener`, jank, lag | Eliminates scroll delay |

---

## 5. Re-render Optimization (MEDIUM)

Reducing unnecessary re-renders minimizes wasted computation and improves UI responsiveness.

| Rule | Description | Keywords | Impact |
|------|-------------|----------|--------|
| [rerender-defer-reads.md](rerender-defer-reads.md) | Don't subscribe to state only used in callbacks | subscribe, selector, callback, zustand, redux, unnecessary re-render | Avoids unnecessary subscriptions |
| [rerender-memo.md](rerender-memo.md) | Extract expensive work into memoized components | `memo`, `React.memo`, expensive, child component, Profiler, highlight | Enables early bailout |
| [rerender-dependencies.md](rerender-dependencies.md) | Use primitive dependencies in effects | `useEffect`, `useMemo`, dependencies, object, primitive, infinite loop | Minimizes effect re-runs |
| [rerender-derived-state.md](rerender-derived-state.md) | Subscribe to derived booleans, not raw values | derived, selector, boolean, `useSyncExternalStore`, threshold | Reduces re-render frequency |
| [rerender-functional-setstate.md](rerender-functional-setstate.md) | Use functional setState for stable callbacks | `setState`, `useCallback`, functional, closure, stale state | Prevents stale closures |
| [rerender-lazy-state-init.md](rerender-lazy-state-init.md) | Pass function to useState for expensive values | `useState`, lazy, initializer, expensive, `JSON.parse` | Avoids wasted computation |
| [rerender-transitions.md](rerender-transitions.md) | Use startTransition for non-urgent updates | `startTransition`, `useTransition`, urgent, interruptible, lag, freeze | Maintains UI responsiveness |

---

## 6. Rendering Performance (MEDIUM)

Optimizing the rendering process reduces the work the browser needs to do.

| Rule | Description | Keywords | Impact |
|------|-------------|----------|--------|
| [rendering-animate-svg-wrapper.md](rendering-animate-svg-wrapper.md) | Animate div wrapper, not SVG element | SVG, animation, wrapper, transform, jank, 60fps, GPU | Enables hardware acceleration |
| [rendering-content-visibility.md](rendering-content-visibility.md) | Use content-visibility for long lists | `content-visibility`, virtualization, long list, CSS, slow scroll | Faster initial render |
| [rendering-hoist-jsx.md](rendering-hoist-jsx.md) | Extract static JSX outside components | static, JSX, hoist, constant, module-level, re-creation | Avoids re-creation |
| [rendering-svg-precision.md](rendering-svg-precision.md) | Reduce SVG coordinate precision | SVG, precision, decimal, file size, SVGO | Reduces file size |
| [rendering-hydration-no-flicker.md](rendering-hydration-no-flicker.md) | Use inline script for client-only data | hydration, `localStorage`, flicker, SSR, theme, dark mode, FOUC | Avoids visual flicker |
| [rendering-activity.md](rendering-activity.md) | Use Activity component for show/hide | `Activity`, show, hide, preserve state, unmount, tab, modal | Preserves state/DOM |
| [rendering-conditional-render.md](rendering-conditional-render.md) | Use ternary, not && for conditionals | `&&`, ternary, conditional, render, falsy, `0`, `NaN` | Prevents rendering 0/NaN |

---

## 7. JavaScript Performance (LOW-MEDIUM)

Micro-optimizations for hot paths can add up to meaningful improvements.

| Rule | Description | Keywords | Impact |
|------|-------------|----------|--------|
| [js-batch-dom-css.md](js-batch-dom-css.md) | Group CSS changes via classes or cssText | DOM, CSS, batch, `cssText`, `classList`, reflow, layout thrashing | Reduces reflows/repaints |
| [js-index-maps.md](js-index-maps.md) | Build Map for repeated lookups | `Map`, lookup, index, O(n), O(1), `find`, slow loop | 1M ops → 2K ops |
| [js-cache-property-access.md](js-cache-property-access.md) | Cache object properties in loops | loop, property, cache, variable, access, hot path | Reduces lookups |
| [js-cache-function-results.md](js-cache-function-results.md) | Cache function results in module-level Map | memoization, cache, function, `Map`, expensive, redundant | Avoids redundant computation |
| [js-cache-storage.md](js-cache-storage.md) | Cache localStorage/sessionStorage reads | `localStorage`, `sessionStorage`, cache, reads, repeated access | Reduces expensive I/O |
| [js-combine-iterations.md](js-combine-iterations.md) | Combine multiple filter/map into one loop | `filter`, `map`, `reduce`, loop, iteration, chained, multiple passes | Reduces iterations |
| [js-length-check-first.md](js-length-check-first.md) | Check array length before expensive comparison | `length`, array, short-circuit, optimization, deep equal | Avoids expensive operations |
| [js-early-exit.md](js-early-exit.md) | Return early from functions | early return, guard clause, nesting, readability, validation | Avoids unnecessary computation |
| [js-hoist-regexp.md](js-hoist-regexp.md) | Hoist RegExp creation outside loops | `RegExp`, loop, hoist, compile, constant, recreation | Avoids recreation |
| [js-min-max-loop.md](js-min-max-loop.md) | Use loop for min/max instead of sort | `Math.min`, `Math.max`, sort, loop, O(n), O(n log n) | O(n) vs O(n log n) |
| [js-set-map-lookups.md](js-set-map-lookups.md) | Use Set/Map for O(1) lookups | `Set`, `Map`, `includes`, `indexOf`, O(1), slow lookup | O(n) → O(1) |
| [js-tosorted-immutable.md](js-tosorted-immutable.md) | Use toSorted() for immutability | `toSorted`, `sort`, immutable, copy, mutation bug, React state | Prevents mutation bugs |

---

## 8. Advanced Patterns (LOW)

Advanced patterns for specific cases that require careful implementation.

| Rule | Description | Keywords | Impact |
|------|-------------|----------|--------|
| [advanced-event-handler-refs.md](advanced-event-handler-refs.md) | Store event handlers in refs | `useRef`, event handler, stable, callback, subscription | Stable subscriptions |
| [advanced-use-latest.md](advanced-use-latest.md) | useLatest for stable callback refs | `useLatest`, ref, stable, callback, closure, effect dependency | Prevents effect re-runs |
