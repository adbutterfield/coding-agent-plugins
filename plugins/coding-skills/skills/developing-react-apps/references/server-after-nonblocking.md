---
title: Use after() for Non-Blocking Operations
impact: MEDIUM
---

## Use after() for Non-Blocking Operations

> **Review:** `await` calls for logging, analytics, or notifications before `return Response`
> **Writing:** When adding logging, analytics, or side effects to Route Handlers or Server Actions
> **Technique:** Use Next.js `after()` to schedule non-critical work after the response is sent

Use Next.js's `after()` to schedule work that should execute after a response is sent. This prevents logging, analytics, and other side effects from blocking the response and improves perceived performance.

> Requires Next.js 15+.

### Fail

```tsx
// ❌ Logging blocks the response - user waits for logging to complete
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // Perform mutation
  await updateDatabase(request)

  // Logging blocks the response
  const userAgent = request.headers.get('user-agent') || 'unknown'
  await logUserAction({ userAgent })

  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}
```

### Pass

```tsx
// ✅ Response sent immediately, logging happens in background
import { after } from 'next/server'
import { headers, cookies } from 'next/headers'
import { logUserAction } from '@/app/utils'

export async function POST(request: Request) {
  // Perform mutation
  await updateDatabase(request)

  // Log after response is sent
  after(async () => {
    const userAgent = (await headers()).get('user-agent') || 'unknown'
    const sessionCookie = (await cookies()).get('session-id')?.value || 'anonymous'

    logUserAction({ sessionCookie, userAgent })
  })

  return new Response(JSON.stringify({ status: 'success' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' }
  })
}
```

### Why This Matters

- Users receive responses faster when non-critical work is deferred
- Logging and analytics should never slow down the user experience
- `after()` runs even if the response fails or redirects, ensuring side effects complete

### When to Apply

- Analytics tracking and event logging
- Audit logging for compliance
- Sending notifications (email, push, webhooks)
- Cache invalidation after mutations
- Cleanup tasks and background processing

### When to Skip

- When the side effect result affects the response (must await)
- Critical operations that must complete before response (payment confirmation)
- When you need error handling tied to the main request
- Next.js versions before 15

### Expected Impact

Faster response times by deferring non-critical operations until after the response is sent. For routes with logging, analytics, or notification side effects, this can reduce perceived latency by 50-200ms or more depending on the duration of the deferred operations.

### See Also

- `async-parallel.md` - Promise.all for parallel async operations
- `async-defer-await.md` - Starting promises before await for parallel execution
