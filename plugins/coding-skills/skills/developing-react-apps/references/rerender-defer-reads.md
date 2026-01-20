---
title: Defer State Reads to Usage Point
impact: MEDIUM
---

## Defer State Reads to Usage Point

> **Review:** `useSearchParams` or `useRouter` where the value is only used inside a callback/handler
> **Writing:** Reading URL params or localStorage only when an action occurs (click, submit, etc.)
> **Technique:** Read from `window.location` or storage APIs directly inside the callback instead of subscribing

Deferring state reads to the point of usage avoids unnecessary subscriptions that cause re-renders whenever the external state changes, even when the component doesn't need to react to those changes.

### Fail

```tsx
// ❌ Subscribes to all searchParams changes, re-renders on every URL change
function ShareButton({ chatId }: { chatId: string }) {
  const searchParams = useSearchParams()

  const handleShare = () => {
    const ref = searchParams.get('ref')
    shareChat(chatId, { ref })
  }

  return <button onClick={handleShare}>Share</button>
}
```

### Pass

```tsx
// ✅ Reads on demand, no subscription, no unnecessary re-renders
function ShareButton({ chatId }: { chatId: string }) {
  const handleShare = () => {
    const params = new URLSearchParams(window.location.search)
    const ref = params.get('ref')
    shareChat(chatId, { ref })
  }

  return <button onClick={handleShare}>Share</button>
}
```

### Why This Matters

- Subscriptions via hooks like `useSearchParams` trigger re-renders on every state change
- Reading on demand inside callbacks avoids these unnecessary re-renders
- Components stay simpler without reactive dependencies they don't need

### When to Apply

- URL search params read only in event handlers
- localStorage/sessionStorage values read only on user actions
- Any external state that doesn't affect the render output directly

### When to Skip

- The component needs to display the current value (e.g., showing current filter in UI)
- The value drives conditional rendering or computed output
- You need to react to changes in real-time (e.g., sync UI with URL state)

### Expected Impact

Avoids unnecessary store subscriptions, preventing re-renders when irrelevant state changes. Components that only read values in event handlers will no longer re-render on every URL or storage change, significantly reducing render frequency in apps with dynamic routing or frequent storage updates.

### See Also

- `rerender-derived-state.md` - Subscribe to derived state instead of continuous values
- `rerender-dependencies.md` - Narrow effect dependencies to reduce re-runs
