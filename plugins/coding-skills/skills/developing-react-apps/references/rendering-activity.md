---
title: Use Activity Component for Show/Hide
impact: MEDIUM
---

## Use Activity Component for Show/Hide

> **Review:** Conditional rendering (`{isOpen && <Component />}`) for expensive components that toggle frequently
> **Writing:** Creating dropdowns, modals, tabs, or panels that show/hide while preserving state
> **Technique:** Use React's `<Activity>` component to hide with `display: none` while preserving state and DOM

Use React's `<Activity>` component to preserve state and DOM for expensive components that frequently toggle visibility. Unlike conditional rendering, Activity keeps the component mounted but hidden.

> Requires React 19.2+

### Fail

```tsx
// ❌ Destroys and recreates ExpensiveMenu on every toggle
function Dropdown({ isOpen }: Props) {
  return (
    <div>
      {isOpen && <ExpensiveMenu />}
    </div>
  )
}
```

### Pass

```tsx
// ✅ Preserves state and DOM, just toggles visibility
import { Activity } from 'react'

function Dropdown({ isOpen }: Props) {
  return (
    <Activity mode={isOpen ? 'visible' : 'hidden'}>
      <ExpensiveMenu />
    </Activity>
  )
}
```

### Why This Matters

- Avoids expensive re-initialization and re-renders when toggling visibility
- Preserves component state (form inputs, scroll position, selection)
- Cleans up Effects when hidden (subscriptions, timers) unlike CSS `display: none`
- Re-activates Effects when shown again

### When to Apply

- Dropdown menus that open/close frequently
- Tab panels where users switch between tabs
- Modal dialogs with complex content
- Sidebars and drawers
- Accordion sections with expensive content
- Any component where state preservation matters

### When to Skip

- Simple, cheap-to-render components
- Components that should reset state when hidden
- React versions below 19.2
- When the hidden component consumes significant memory and should be unmounted
- Components that are rarely toggled (one-time modals)

### Expected Impact

Preserves component state and DOM when hiding and showing UI, avoiding expensive re-initialization. This is particularly valuable for components with complex state (form inputs, scroll positions) or heavy initialization costs (data fetching, subscriptions), resulting in near-instant toggle transitions.

### See Also

- `rendering-content-visibility.md` - CSS-based visibility optimization for lists
- `rendering-conditional-render.md` - Safe conditional rendering patterns
