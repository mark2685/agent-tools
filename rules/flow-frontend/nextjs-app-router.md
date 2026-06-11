---
globs:
  - "apps/**/*.tsx"
  - "apps/**/*.ts"
---

# Next.js App Router Patterns

Rules for working with the Next.js App Router (Next 16) — both apps, `flow-global` and `flow-factory`, follow these conventions.

## Directives

### `'use client'`
- Add only when the component uses hooks, event handlers, browser APIs, or class components
- Place at the top of the file, before all imports
- Push as deep in the component tree as possible — wrap the interactive leaf, not the page
- A `'use client'` file makes all its imports client-side too — be intentional about the boundary

### `'use server'`
- Used in per-domain server action files under `_lib/actions/` (e.g. `_lib/actions/<domain>.actions.ts`)
- All server actions must call RPC clients via `@hadrian-mtv/connect-server-actions`
- Authenticate server actions like public API routes — never rely solely on middleware

## RSC Boundaries

### Detecting Boundary Violations
A component needs `'use client'` if it:
- Calls `useState`, `useEffect`, `useReducer`, `useRef`, or any custom hook using them
- Uses event handlers (`onClick`, `onChange`, `onSubmit`, etc.)
- Accesses browser APIs (`window`, `document`, `localStorage`)
- Uses `use()` for context consumption

A component should stay as a server component if it:
- Only renders HTML/JSX with no interactivity
- Fetches data via server actions or direct DB/RPC calls
- Passes data down to client components as props

### Pattern: Server Wrapper + Client Leaf
```tsx
// page.tsx (server component — no directive)
export default async function OrderPage({ params }: Props) {
  const order = await getOrder(params.id)
  return <OrderDetailsClient order={order} />
}

// OrderDetailsClient.tsx
'use client'
export function OrderDetailsClient({ order }: { order: Order }) {
  const [isEditing, setIsEditing] = useState(false)
  // interactive logic here
}
```

### Serialize Only What Clients Need
The RSC boundary serializes every property of an object passed to a client component. Pass only the fields the client uses, not the whole record.

```tsx
// Incorrect — serializes the entire order
<OrderDetailsClient order={order} />

// Correct — only the fields the client reads
<OrderDetailsClient id={order.id} status={order.status} total={order.total} />
```

## Data Fetching

### Prevent Waterfalls
- Parallelize independent data fetches with `Promise.all()` at the page/layout level
- Use component-level data fetching with Suspense boundaries for independent UI sections
- Defer `await` until the data is actually needed

```tsx
// Incorrect — sequential
export default async function Page({ params }: Props) {
  const order = await getOrder(params.id)
  const customer = await getCustomer(order.customerId) // depends on order
  const analytics = await getAnalytics(params.id)      // does NOT depend on order
  return <OrderView order={order} customer={customer} analytics={analytics} />
}

// Correct — parallel where possible
export default async function Page({ params }: Props) {
  const [order, analytics] = await Promise.all([
    getOrder(params.id),
    getAnalytics(params.id),
  ])
  const customer = await getCustomer(order.customerId)
  return <OrderView order={order} customer={customer} analytics={analytics} />
}
```

### Never Fetch in `useEffect`
- Do not fetch data in `useEffect` + `useState`. It re-introduces the waterfall, runs client-side, and skips Suspense/error boundaries.
- Server-side fetch (RSC / server action) is the default. When data must load on the client, use TanStack Query — never a hand-rolled effect.

### TanStack Query for Client-Side State
- Use TanStack Query v5 for client-side server state; mutations call server actions, not RPC clients directly
- Query definitions, key shape, and invalidation follow the `tanstack-query` rule (installed at `.claude/rules/tanstack-query.md`)

### Caching with Cache Components (Next 16 — not enabled here yet)

Next 16 adds explicit caching via the `'use cache'` directive plus `cacheLife()` (gated by the `cacheComponents` config), superseding experimental PPR. This codebase has **not** enabled `cacheComponents`, so do not add `'use cache'` directives today — server components render dynamically by default. When the apps adopt it, cache at the data-fetch or component boundary and set freshness with `cacheLife('minutes' | 'hours' | …)`; the function's arguments become part of the cache key. Reference: <https://nextjs.org/docs/app/getting-started/caching>.

## Error Handling

### File Conventions
- `error.tsx` — must be `'use client'`, receives `error` and `reset` props
- `loading.tsx` — server component skeleton shown during async data fetching
- `not-found.tsx` — rendered when `notFound()` is called

### Error Boundary Pattern
```tsx
'use client'
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <Text>Something went wrong loading this page.</Text>
      <Button type="button" onClick={reset}>Try again</Button>
    </div>
  )
}
```

## Hydration Errors

Common causes and fixes (none of these is a reason to move data fetching into the client):
- **Browser extensions** inject DOM nodes — wrap affected components in `Suspense` with a client fallback
- **Date/time rendering** differs server vs. client — format on the server, or add `suppressHydrationWarning` on the single text node that legitimately differs
- **Conditional rendering on `typeof window`** — render the same markup on both passes; gate the browser-only branch behind a mount flag rather than fetching or branching during render
- **Invalid HTML nesting** (e.g., `<p>` inside `<p>`, `<div>` inside `<p>`) — fix the markup

To track down a hydration mismatch systematically, use the `systematic-debug` skill — do not inline the debugging steps here.

## Async Patterns (Next 16)

- `params` and `searchParams` are now Promises — always `await` them:
  ```tsx
  export default async function Page({ params }: { params: Promise<{ id: string }> }) {
    const { id } = await params
  }
  ```
- `cookies()` and `headers()` are async — `await` before accessing
- Dynamic APIs (`cookies`, `headers`, `searchParams`) opt the route into dynamic rendering

## Route Organization

- Co-locate domain logic in `_lib/` directories adjacent to the page
- `_lib/actions/` — per-domain server action files (`<domain>.actions.ts`), not a single `actions.ts`
- `_lib/queries.ts` for TanStack Query definitions and hooks (conventions in the `tanstack-query` rule, installed at `.claude/rules/tanstack-query.md`)
- `_lib/components/` for page-specific components
- Shared cross-domain logic goes in `app/_lib/` or `lib/` at the app root

## Boundaries

- No flow-global cross-cluster fetching: `flow-global` must not reach into a factory cluster's data directly. Cross-cluster data is aggregated on the backend and exposed as a single API — not stitched together with per-cluster requests from the frontend.
