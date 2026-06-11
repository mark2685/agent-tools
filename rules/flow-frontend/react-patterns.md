---
globs:
  - "apps/**/*.tsx"
  - "packages/**/*.tsx"
---

# React Patterns

Rules for building maintainable, performant React components in this monorepo (React 19, Next 16). Adapted from Vercel's React best practices and composition patterns, tailored to our stack.

**React Compiler is not enabled here.** Memoization is your responsibility, but it is an escape hatch — reach for the structural fix first (derive inline, place state correctly, compose) and memoize only the specific cases called out below.

## Component Architecture

### Composition Over Boolean Props

Do not add boolean props like `isEditing`, `isCompact`, `showHeader` to customize component behavior. Each boolean doubles possible states. Use composition instead.

**Incorrect:**
```tsx
<OrderDetails isEditing={false} isCompact showActions={false} />
```

**Correct:** Create explicit variant components that compose shared internals:
```tsx
// Each variant is explicit about what it renders; an editable variant
// composes the same internals with EditableLineItems + Actions instead.
function OrderDetailsReadOnly({ orderId }: { orderId: string }) {
  return (
    <OrderDetails.Frame>
      <OrderDetails.Header />
      <OrderDetails.LineItems />
      <OrderDetails.Summary />
    </OrderDetails.Frame>
  )
}
```

### Compound Components with Shared Context

Structure complex components as compound components — subcomponents access shared state via context, not props, and consume it with `use()` (React 19). Export the parts as one object:

```tsx
const Composer = {
  Provider: ComposerProvider, // owns state management; renders <ComposerContext value={…}>
  Frame: ComposerFrame,
  Input: ComposerInput,       // const { state, actions } = use(ComposerContext)
  Submit: ComposerSubmit,
}
```

### State Management Decoupling

- Lift state into dedicated provider components so sibling components outside the main UI can access it
- Providers are the only place that knows how state is managed (Zustand, TanStack Query, useState)
- UI components consume the context interface — they don't know the state implementation
- Different providers can implement the same interface for different use cases (a `useState` provider for an ephemeral form, a TanStack Query provider for server-synced state)

### Never Define Components Inside Components

A component defined inside another is a new component type on every render. React remounts it — destroying its state and DOM, re-running its effects. The usual reason is to close over a parent variable; pass it as a prop instead.

### Pass Children as Props to Isolate Re-renders

A component that owns frequently-changing state should take static subtrees as `children` rather than rendering them itself. Children passed in are created by the parent and don't re-render when the stateful component updates.

```tsx
// Incorrect — <ExpensiveTree /> re-renders on every color change
function Layout() {
  const [color, setColor] = useState('red')
  return <div style={{ color }}><ExpensiveTree /></div>
}

// Correct — ExpensiveTree is a stable child, unaffected by color state
function ColorContainer({ children }: { children: React.ReactNode }) {
  const [color, setColor] = useState('red')
  return <div style={{ color }}>{children}</div>
}
// <ColorContainer><ExpensiveTree /></ColorContainer>
```

### Stable Keys — `item.id`, Never the Array Index

List items must be keyed by a stable identity field (`item.id`, `fileHash`, a proto resource name). An array index as a key tears state and DOM when the list reorders, filters, or has items inserted — inputs keep stale values, the wrong row animates. This is the most common correctness bug in our review history.

```tsx
// Incorrect — index breaks on reorder/insert/filter
{items.map((item, i) => <Row key={i} item={item} />)}

// Correct — stable identity
{items.map((item) => <Row key={item.id} item={item} />)}
```

### Refs: `ref` as a Prop (React 19)

React 19 makes `ref` a regular prop — new function components take `{ ref, ...props }` directly and forward it to the DOM node, with no `forwardRef` wrapper. `forwardRef` still works, so existing wrappers (`Input`, `Select`, etc.) need not be rewritten — migrate them incrementally when you next touch them. Reserve `useImperativeHandle` for exposing a curated method surface, not the raw node.

## Performance

Reach for the structural fix before memoizing. When you do profile, use the React DevTools Profiler and Chrome Performance panel; tie decisions to Core Web Vitals — LCP < 2.5s, INP < 200ms, CLS < 0.1. (Data-fetch parallelization — `Promise.all`, avoiding waterfalls — is owned by the `nextjs-app-router` rule, installed at `.claude/rules/nextjs-app-router.md`.)

### Eliminate Barrel File Imports

Never import from barrel files (index.ts re-exports). Import directly from the source module to avoid pulling thousands of unused modules into the bundle.

```tsx
// Incorrect
import { Button, Dialog } from '@hadrian-mtv/ui-toolkit'

// Correct
import { Button } from '@hadrian-mtv/ui-toolkit/button'
import { Dialog } from '@hadrian-mtv/ui-toolkit/dialog'
```

### Derive State During Render

Do not store values that can be computed from existing state or props, and never sync them in an effect. Derive them inline during render. A `useState` + `useEffect` pair for derived data causes an extra render and lets the copy drift out of sync.

```tsx
// Incorrect — stored derived state, double render, drift risk
const [filteredItems, setFilteredItems] = useState([])
useEffect(() => { setFilteredItems(items.filter(i => i.active)) }, [items])

// Correct — derived inline during render
const filteredItems = items.filter(i => i.active)
```

`useMemo` is an **escape hatch**, not the default — the compiler is not enabled, so an inline derivation recomputes on every render. Memoize only when the computation is genuinely expensive (profile first; rule of thumb >1ms), when you need a stable reference to feed a dependency array, or when a memoized child relies on referential equality. For a cheap filter or string concat, plain inline is correct.

```tsx
// Escape hatch — expensive enough to justify, and dependencies are correct
const searchResults = useMemo(() => fuzzyRank(items, query), [items, query])
```

When a memo bundles independent steps, split it so a change to one dependency doesn't recompute the others.

```tsx
const filtered = useMemo(() => products.filter(p => p.category === category), [products, category])
const sorted = useMemo(() => filtered.toSorted(byPrice(sortOrder)), [filtered, sortOrder])
```

For expensive `useState` initializers, pass a function so it runs once, not on every render: `useState(() => buildIndex(items))`.

### Use Immutable Operations for React State

Use `.toSorted()`, `.toReversed()`, and spread/map instead of mutating `.sort()`, `.reverse()`, `.splice()` on React state arrays. `.sort()` mutates in place, which silently mutates a prop or state array and breaks React's read-only model.

### Code-split Heavy Client Components

Lazy-load large client components not needed on first paint with `next/dynamic` — this is the single biggest TTI/LCP lever for heavy widgets (editors, charts, grids like ag-grid).

```tsx
import dynamic from 'next/dynamic'

const SpecEditor = dynamic(() => import('./spec-editor').then(m => m.SpecEditor), { ssr: false })
```

Defer non-critical third-party libraries (analytics, logging) the same way so they load after hydration instead of blocking the initial bundle.

## Concurrent Features

Decide between these by profiling — reach for them when input feels laggy or a list update blocks typing, not preemptively.

- **`useTransition`** when *you* trigger a state update and want a built-in `isPending` flag. Mark the heavy state update non-urgent so the urgent update (the input value, the click) commits immediately. Prefer it over a hand-managed `isLoading` `useState`.
- **`useDeferredValue`** when you *receive* a value (a prop, a controlled input) and want the expensive render that consumes it to lag without blocking. Wrap the consuming computation in `useMemo` keyed on the deferred value, or it still runs every render.

```tsx
// useTransition — you own the update
const [isPending, startTransition] = useTransition()
const onFilter = (next: string) => { setQuery(next); startTransition(() => setRows(filter(allRows, next))) }

// useDeferredValue — value comes from outside
const deferredQuery = useDeferredValue(query)
const results = useMemo(() => fuzzyRank(items, deferredQuery), [items, deferredQuery])
```

Both target INP < 200ms by keeping the urgent interaction off the critical path. For route-level weight, split at the route boundary (App Router segments load per-route) rather than shipping everything in one chunk.

## Server-side Deduplication

For data fetched more than once per request on the server (the current user, a shared lookup), wrap the fetcher in React's `cache()` so it runs once per request. Pass primitive arguments — `cache()` keys on `Object.is`, so an inline object is always a miss.

```ts
import { cache } from 'react'
export const getCurrentUser = cache(async () => { /* … */ })
```

## UI Verification

After building or modifying UI, check it against the `design-reviewer` agent's UX-defect checklist (`.claude/agents/design-reviewer.md`) — it is the canonical list for states, keyboard access, and layout defects. Also confirm no console errors or React warnings in the browser, and that the component renders correctly with realistic data (not just happy-path short strings).

## Rendering and Server Components

Server-vs-client boundaries, the server-wrapper/client-leaf pattern, and RSC prop serialization live in the `nextjs-app-router` rule (installed at `.claude/rules/nextjs-app-router.md`). Follow it for where to place `'use client'` and what to serialize across the boundary.
