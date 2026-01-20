---
title: Use Explicit Conditional Rendering
impact: LOW
---

## Use Explicit Conditional Rendering

> **Review:** `{count &&` or `{value &&` where the variable is a number
> **Writing:** Rendering content conditionally based on numeric values or potentially falsy data
> **Technique:** Use explicit boolean checks or ternary operators instead of `&&`

Use explicit ternary operators (`? :`) or boolean coercion (`!!` or `> 0`) instead of `&&` for conditional rendering when the condition can be `0`, `NaN`, or other falsy values that React renders as text.

### Fail

```tsx
// ❌ Renders "0" when count is 0
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count && <span className="badge">{count}</span>}
    </div>
  )
}

// When count = 0, renders: <div>0</div>
// When count = 5, renders: <div><span class="badge">5</span></div>
```

### Pass

```tsx
// ✅ Renders nothing when count is 0
function Badge({ count }: { count: number }) {
  return (
    <div>
      {count > 0 ? <span className="badge">{count}</span> : null}
    </div>
  )
}

// When count = 0, renders: <div></div>
// When count = 5, renders: <div><span class="badge">5</span></div>
```

```tsx
// ✅ Alternative: Boolean coercion
function Badge({ count }: { count: number }) {
  return (
    <div>
      {!!count && <span className="badge">{count}</span>}
    </div>
  )
}
```

### Why This Matters

- JavaScript `&&` returns the first falsy value, not `false`
- `0 && <Component />` returns `0`, which React renders as text "0"
- `NaN && <Component />` returns `NaN`, which React renders as text "NaN"
- This is a common source of UI bugs that are easy to miss in testing

### When to Apply

- Rendering based on numeric counts (badges, notifications, quantities)
- Rendering based on array length (`items.length && ...`)
- Any condition that might evaluate to `0` or `NaN`
- Data that could be `undefined`, `null`, `0`, or empty string

### When to Skip

- Conditions that are strictly boolean (`isOpen && ...`)
- String conditions where empty string rendering nothing is acceptable
- Object/array existence checks (`user && ...` where user is object or undefined)

### Expected Impact

Prevents accidentally rendering falsy values like `0` or `NaN` as visible text in the UI. This eliminates a common class of subtle bugs where notification badges show "0" or numeric displays show "NaN" instead of rendering nothing.

### See Also

- `rendering-activity.md` - Alternative to conditional rendering for visibility
