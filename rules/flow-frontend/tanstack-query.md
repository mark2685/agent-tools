---
globs:
  - "apps/**/*.ts"
  - "apps/**/*.tsx"
---

# TanStack Query v5 Conventions

Server state goes through TanStack Query v5 (`@tanstack/react-query`). Never hand-manage `loading` / `error` / `data` with `useEffect` + `useState` — use `useQuery` / `useSuspenseQuery` / `useMutation`. This rule is the single owner of query and query-key conventions; other rules and agents reference it rather than restating.

## Adoption status

`queryOptions()` factories are the **target** standard, not current practice. The codebase today defines queries as `get*QueryKey()` helpers plus `UseQueryOptions`-typed objects in `_lib/queries.ts` files (e.g. `apps/flow-global/app/orders/_lib/queries.ts`). Apply `queryOptions()` to new query code; do **not** flag existing `get*QueryKey()`-style code in reviews; migrate opportunistically when touching legacy queries.

## Query options factories

Define every query as a `queryOptions()` factory, not an inline object at the call site. This gives the query a single typed home that both `useQuery` and prefetch/`ensureQueryData` calls share.

```ts
export const orderQueries = {
  all: () => ['orders'] as const,
  lists: () => [...orderQueries.all(), 'list'] as const,
  list: (siteCode: string) =>
    queryOptions({
      queryKey: [...orderQueries.lists(), { siteCode }],
      queryFn: () => fetchOrders(siteCode),
    }),
  detail: (orderId: string) =>
    queryOptions({
      queryKey: [...orderQueries.all(), 'detail', orderId],
      queryFn: () => fetchOrder(orderId),
    }),
};
```

## Hierarchical keys

Keys are arrays ordered general → specific: `[domain, scope, params]`. Build them through the factory so they stay consistent across the team — never hand-write a key string at a call site. Consistent hierarchy makes partial invalidation work: `invalidateQueries({ queryKey: orderQueries.lists() })` clears every list without touching detail caches.

- Serializable params go in a trailing object: `[...lists(), { siteCode, status }]`.
- One factory per domain; do not invent a parallel key shape elsewhere for the same data.

## Mutations: separate side effects from the request

Keep `mutationFn` to the request only. Put cache updates and notifications in `onSuccess` / `onError` — do not stuff invalidation, toasts, or navigation inside `mutationFn`.

```ts
useMutation({
  mutationFn: (input: UpdateOrderRequired) => updateOrder(input),
  onSuccess: () => queryClient.invalidateQueries({ queryKey: orderQueries.all() }),
  onError: (error) => toast.error(getErrorMessage(error)),
});
```

Surface the why: render the failure with `getErrorMessage(error)` rather than a generic string, so the user and logs see the real cause.

## No N+1 fetching

Never fire one query per row. A request inside a `.map` / loop, or a child component that fetches its own slice of a parent list, is an N+1 pattern — aggregate on the backend and fetch once. If the backend lacks a batch endpoint, that is a backend change, not a client workaround.
