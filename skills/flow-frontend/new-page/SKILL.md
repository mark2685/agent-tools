---
name: new-page
description: Scaffold a new page in flow-global or flow-factory following all project conventions. Use when user asks to "create a page", "add a new route", "scaffold a page", or "/new-page".
---

# New Page Scaffold

Create a new Next.js App Router page with all conventions followed.

## Steps

1. **Ask** which app (`flow-global` or `flow-factory`) and the route path (e.g., `/orders/[orderId]/details`)

2. **Determine domain** — the top-level route segment determines the domain:
   - **flow-global**: `orders`, `quoting`, `reference-services`
   - **flow-factory**: `crib`, `factory-execution`, `inventory`, `supply-chain`, `task-management`, `tool-prep`

3. **Create the page** at `apps/<app>/app/<route>/page.tsx`:
   - Default to a **server component** (no `'use client'` directive)
   - Use TypeScript with proper param types for dynamic segments
   - Follow existing page patterns in the same domain for consistency

4. **Create co-located files** as needed:
   - `_lib/actions.ts` — server actions (`'use server'`) using `@hadrian-mtv/connect-server-actions`
   - `_lib/queries.ts` — TanStack Query hooks if client-side data fetching is needed
   - `loading.tsx` — loading skeleton if the page does async data fetching
   - `error.tsx` — error boundary (must be `'use client'`)

5. **Regenerate route types**: Run `pnpm build:typesafe-url`

6. **Verify navigation types**: Confirm the new route is available in `@hadrian-mtv/flow-navigation` types

7. **Check domain boundaries**: Ensure all imports respect `eslint-plugin-boundaries` rules:
   - Only import from allowed sibling domains + shared layers (`app/_lib/`, `lib/`, `utils/`)
   - Use `FlowLink`/`FlowLinkButton` for all navigation — never `next/link`
   - Use `@hadrian-mtv/flow-logger` for logging — never `console.*`

8. **Verify** with `pnpm lint:tsc` that the page compiles cleanly
