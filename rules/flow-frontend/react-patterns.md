# React Patterns

Rules for building maintainable, performant React components in this monorepo. Adapted from Vercel's composition patterns and React best practices, tailored to our stack.

## Component Architecture

### Composition Over Boolean Props

Do not add boolean props like `isEditing`, `isCompact`, `showHeader` to customize component behavior. Each boolean doubles possible states. Use composition instead.

**Incorrect:**
```tsx
<OrderDetails isEditing={false} isCompact showActions={false} />
```

**Correct:** Create explicit variant components that compose shared internals:
```tsx
// Each variant is explicit about what it renders
function OrderDetailsReadOnly({ orderId }: { orderId: string }) {
  return (
    <OrderDetails.Frame>
      <OrderDetails.Header />
      <OrderDetails.LineItems />
      <OrderDetails.Summary />
    </OrderDetails.Frame>
  )
}

function OrderDetailsEditable({ orderId }: { orderId: string }) {
  return (
    <OrderDetails.Frame>
      <OrderDetails.Header />
      <OrderDetails.EditableLineItems />
      <OrderDetails.Actions>
        <OrderDetails.Cancel />
        <OrderDetails.Save />
      </OrderDetails.Actions>
    </OrderDetails.Frame>
  )
}
```

### Compound Components with Shared Context

Structure complex components as compound components. Subcomponents access shared state via context, not props. Use `use()` (React 19) for context consumption.

```tsx
const ComposerContext = createContext<ComposerContextValue | null>(null)

// Provider handles all state management details
function ComposerProvider({ children, state, actions }: ProviderProps) {
  return <ComposerContext value={{ state, actions }}>{children}</ComposerContext>
}

// UI components consume the interface, not the implementation
function ComposerInput() {
  const { state, actions: { update } } = use(ComposerContext)
  return <Input value={state.input} onChange={(text) => update(s => ({ ...s, input: text }))} />
}

// Export as compound component
const Composer = {
  Provider: ComposerProvider,
  Frame: ComposerFrame,
  Input: ComposerInput,
  Submit: ComposerSubmit,
}
```

### State Management Decoupling

- Lift state into dedicated provider components so sibling components outside the main UI can access it
- Providers are the only place that knows how state is managed (Zustand, TanStack Query, useState)
- UI components consume the context interface — they don't know the state implementation
- Different providers can implement the same interface for different use cases

```tsx
// Local state for ephemeral forms
function ForwardProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState(initialState)
  return <Composer.Provider state={state} actions={{ update: setState }}>{children}</Composer.Provider>
}

// TanStack Query for server-synced state
function ChannelProvider({ channelId, children }: Props) {
  const { data, mutate } = useChannelQuery(channelId)
  return <Composer.Provider state={data} actions={{ update: mutate }}>{children}</Composer.Provider>
}
```

## Performance

### Eliminate Barrel File Imports

Never import from barrel files (index.ts re-exports). Import directly from the source module to avoid pulling thousands of unused modules into the bundle.

```tsx
// Incorrect
import { Button, Dialog } from '@hadrian-mtv/ui-toolkit'

// Correct
import { Button } from '@hadrian-mtv/ui-toolkit/button'
import { Dialog } from '@hadrian-mtv/ui-toolkit/dialog'
```

### Parallelize Independent Fetches

Never `await` sequential independent operations. Use `Promise.all()` or component composition with Suspense.

```tsx
// Incorrect — sequential waterfall
const order = await getOrder(id)
const customer = await getCustomer(customerId)

// Correct — parallel
const [order, customer] = await Promise.all([getOrder(id), getCustomer(customerId)])
```

### Derive State During Render

Do not store values that can be computed from existing state or props. Derive them during render instead.

```tsx
// Incorrect — stored derived state
const [filteredItems, setFilteredItems] = useState([])
useEffect(() => { setFilteredItems(items.filter(i => i.active)) }, [items])

// Correct — derived during render
const filteredItems = useMemo(() => items.filter(i => i.active), [items])
```

### Avoid Stale Closures

Use functional `setState` updates in callbacks and effects to prevent stale closure bugs.

```tsx
// Incorrect — captures stale count
onClick={() => setCount(count + 1)}

// Correct — always uses current value
onClick={() => setCount(prev => prev + 1)}
```

### Use Immutable Operations for React State

Use `.toSorted()`, `.toReversed()`, and spread/map instead of mutating `.sort()`, `.reverse()`, `.splice()` on React state arrays.

## UI Verification

After building or modifying UI, verify:

- All interactive elements are keyboard accessible (Tab through the page)
- Loading, error, and empty states are all handled
- Design tokens used consistently (no hardcoded neutrals)
- No console errors or React warnings in the browser
- Component renders correctly with realistic data (not just happy-path short strings)

## Rendering

### Minimize Client Components

- Server components by default — only add `'use client'` when the component uses hooks, event handlers, or browser APIs
- Push `'use client'` boundaries as deep as possible — wrap only the interactive leaf, not the entire tree
- Pass server-fetched data down as props rather than re-fetching on the client

### Serialize Only What Clients Need

When passing data across the server/client boundary, only include the fields the client actually uses. The RSC boundary serializes all object properties into strings.

```tsx
// Incorrect — serializes entire order object
<OrderCard order={order} />

// Correct — only pass needed fields
<OrderCard id={order.id} status={order.status} total={order.total} />
```
