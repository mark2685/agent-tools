---
name: new-page
description: Scaffolds a new page in flow-global or flow-factory following all project conventions. Use when user asks to "create a page", "add a new route", "scaffold a page", or "/new-page".
---

# New Page Scaffold

Create a new Next.js App Router page with all conventions followed.

## Steps

1. **Ask** which app (`flow-global` or `flow-factory`) and the route path (e.g., `/orders/[orderId]/details`)

2. **Determine domain** — the top-level route segment determines the domain. List the live
   domains with `ls apps/<app>/app/` — do not rely on a memorized list; domains drift. Every
   directory is a domain except `_`-prefixed dirs (`_lib`, `_fonts`), `api`, `healthcheck`,
   and `post-login`.

3. **Create the page** at `apps/<app>/app/<route>/page.tsx`:
   - Default to a **server component** (no `'use client'` directive)
   - Use TypeScript with proper param types for dynamic segments
   - Follow existing page patterns in the same domain for consistency

4. **Fetch data on the server by default.** The page (a server component) awaits its data
   directly. Use `Promise.all` for independent fetches so they don't waterfall. Reserve
   client-side TanStack Query for genuinely client-driven data — mutations, polling, infinite
   scroll, or state that reacts to user interaction — not for the page's initial load.

5. **Push `'use client'` to the smallest leaf.** Keep the page and as many children as possible
   as server components; mark only the interactive child (the one with hooks/handlers) as a
   client component, and pass server-rendered children into it via props/`children` rather than
   making a whole subtree client.

6. **Create co-located files** as needed:
   - `_lib/actions/` — server actions (`'use server'`) using `@hadrian-mtv/connect-server-actions`.
     Put actions in the `_lib/actions` folder, not a single `_lib/actions.ts`.
   - `_lib/queries.ts` — **only if** client-side fetching is actually needed. Define
     `queryOptions()` factories with hierarchical keys built through the domain's key factory
     per the `tanstack-query` rule (installed at `.claude/rules/tanstack-query.md`) — never
     hand-write a literal key.
   - `loading.tsx` — loading skeleton if the page does async data fetching
   - `error.tsx` — error boundary (must be `'use client'`)

7. **Regenerate route types**: Run `pnpm build:typesafe-url`

8. **Verify navigation types**: Confirm the new route is available in `@hadrian-mtv/flow-navigation` types

9. **Check domain boundaries**: Ensure all imports respect `eslint-plugin-boundaries` rules:
   - Only import from allowed sibling domains + shared layers (`app/_lib/`, `lib/`, `utils/`)
   - Use `FlowLink`/`FlowLinkButton` for all navigation — never `next/link`
   - Use `@hadrian-mtv/flow-logger` for logging — never `console.*`

10. **Verify** with `pnpm lint:tsc` that the page compiles cleanly

## Gotchas

- Domain lists drift — always read the filesystem. A hard-coded list in this skill was missing
  four live domains within months of being written.
