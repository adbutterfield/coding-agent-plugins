---
title: CSS content-visibility for Long Lists
impact: HIGH
---

## CSS content-visibility for Long Lists

> **Review:** `.map()` rendering 100+ items without `content-visibility` CSS
> **Writing:** Rendering long scrollable lists, feeds, tables, or message threads
> **Technique:** Apply `content-visibility: auto` with `contain-intrinsic-size` to list items

Use CSS `content-visibility: auto` to defer rendering of off-screen items. The browser skips layout and paint for hidden items, dramatically improving initial render time for long lists.

### Fail

```tsx
// ❌ Browser renders all 1000 items on initial load
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="overflow-y-auto h-screen">
      {messages.map(msg => (
        <div key={msg.id} className="message-item">
          <Avatar user={msg.author} />
          <div>{msg.content}</div>
        </div>
      ))}
    </div>
  )
}
```

```css
/* ❌ No content-visibility optimization */
.message-item {
  padding: 16px;
}
```

### Pass

```tsx
// ✅ Browser only renders visible items (~10), defers the rest
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div className="overflow-y-auto h-screen">
      {messages.map(msg => (
        <div key={msg.id} className="message-item">
          <Avatar user={msg.author} />
          <div>{msg.content}</div>
        </div>
      ))}
    </div>
  )
}
```

```css
/* ✅ content-visibility defers off-screen rendering */
.message-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px; /* estimated height for scroll calculation */
}
```

### Why This Matters

- For 1000 messages, browser skips layout/paint for ~990 off-screen items
- Can achieve 10x faster initial render on long lists
- Native browser optimization with no JavaScript overhead

### When to Apply

- Chat/message lists with 100+ items
- Social media feeds
- Large data tables
- Comment threads
- Any scrollable list where items have predictable heights

### When to Skip

- Short lists under 50 items (overhead not worth it)
- Lists where item height varies wildly and can't be estimated
- When you need all items measured for features like "scroll to item"
- Virtual scrolling is already implemented (redundant)

### Expected Impact

Faster initial render by skipping layout and paint calculations for off-screen content. The performance improvement scales with list length—for a 1000-item list, you may see up to 10x faster initial render times since the browser only processes the ~10 visible items.

### See Also

- `rendering-activity.md` - Hide UI while preserving state
- `rendering-hoist-jsx.md` - Hoist static elements in lists
