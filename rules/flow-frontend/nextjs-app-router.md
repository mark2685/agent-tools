# Next.js App Router Patterns

Rules for working with Next.js 15 App Router in this monorepo. Both `flow-global` (port 3000) and `flow-factory` (port 3001) follow these conventions.

## Directives

### `'use client'`
- Add only when the component uses hooks, event handlers, browser APIs, or class components
- Place at the top of the file, before all imports
- Push as deep in the component tree as possible — wrap the interactive leaf, not the page
- A `'use client'` file makes all its imports client-side too — be intentional about the boundary

### `'use server'`
- Used in server action files (`app/_lib/actions.ts`)
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

### TanStack Query for Client-Side State
- Use TanStack Query v5 for all server state on the client
- Mutations should call server actions, not RPC clients directly
- Invalidate related queries after mutations

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

Common causes and fixes:
- **Browser extensions** inject DOM nodes — wrap affected components in `Suspense` with a client fallback
- **Date/time rendering** differs server vs. client — format dates on the server or suppress hydration with `useEffect`
- **Conditional rendering on `typeof window`** — use `useEffect` + state instead
- **Invalid HTML nesting** (e.g., `<p>` inside `<p>`, `<div>` inside `<p>`) — fix the markup

## Async Patterns (Next.js 15+)

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
- `_lib/actions.ts` for server actions
- `_lib/queries.ts` for TanStack Query hooks
- `_lib/components/` for page-specific components
- Shared cross-domain logic goes in `app/_lib/` or `lib/` at the app root
