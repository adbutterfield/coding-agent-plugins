---
title: Early Return from Functions
impact: LOW-MEDIUM
---

## Early Return from Functions

> **Review:** Loops that continue processing after result is known; flag variables like `hasError`
> **Writing:** When validating collections or searching for a condition
> **Technique:** Return immediately when the answer is determined

Avoids unnecessary computation by exiting as soon as the result is known. Particularly valuable in validation functions and search operations.

### Fail

```typescript
// ❌ Processes all items even after finding an error
function validateUsers(users: User[]) {
  let hasError = false
  let errorMessage = ''

  for (const user of users) {
    if (!user.email) {
      hasError = true
      errorMessage = 'Email required'
    }
    if (!user.name) {
      hasError = true
      errorMessage = 'Name required'
    }
  }

  return hasError ? { valid: false, error: errorMessage } : { valid: true }
}
```

### Pass

```typescript
// ✅ Returns immediately on first error found
function validateUsers(users: User[]) {
  for (const user of users) {
    if (!user.email) {
      return { valid: false, error: 'Email required' }
    }
    if (!user.name) {
      return { valid: false, error: 'Name required' }
    }
  }

  return { valid: true }
}
```

### Why This Matters

- Reduces average-case runtime by exiting early
- Eliminates flag variables that obscure control flow
- Makes the "happy path" and error conditions clearer

### When to Apply

- Validation functions that check multiple conditions
- Search operations looking for the first match
- Any loop where the final result can be determined before completing all iterations

### When to Skip

- When you need to collect all errors (e.g., form validation showing all issues)
- When side effects must run for every item regardless of result
- When the function must process all items for metrics or logging

### Expected Impact

Avoids unnecessary computation by returning early when preconditions fail. This pattern reduces average-case runtime and eliminates flag variables that obscure control flow, making code both faster and more readable.

### See Also

- `js-length-check-first.md` - Specific early-exit pattern for array comparisons
