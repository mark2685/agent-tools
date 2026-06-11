---
name: new-component
description: Scaffolds a new React component following project conventions. Use when user asks to "create a component", "add a new component", or "/new-component".
---

# /new-component

Scaffold a new React component following project conventions.

## Steps

1. Ask the user:
   - Component name
   - Which app: `flow-factory` or `flow-global` (or a shared package in `packages/`)
   - Feature directory (e.g., `app/orders/_lib/components/`)

2. **Before creating anything, search for something to reuse or extend.** Hand-rolling a
   primitive that already exists is the most-flagged review defect. Search the full shared tier,
   not just one package:
   - `packages/ui-toolkit/src/` — primitives (Button, Alert, Link, inputs)
   - `packages/shared-utils/`, `packages/part-utils/` and other domain `*-utils` packages
   - any domain toolkit/UI package matching the feature (e.g. `part-toolkit`,
     `factory-execution-ui` — list them with `ls packages/`)
   - the target package's source directly — its `package.json` `exports` map and the `src/`
     files it points to show what already exists.
   If a close match exists, extend or compose it instead of starting from scratch.

3. Create the component file:
   - Add `'use client'` directive only if it uses hooks, event handlers, or browser APIs
   - Import UI primitives from `@hadrian-mtv/ui-toolkit/*`
   - Use `@hadrian-mtv/classname-variants` for variant styling
   - Use `cnMerge` from `@hadrian-mtv/ui-toolkit` for conditional classes
   - Use Tailwind design tokens (`surface-*`, `content-*`, `border-*`) not hardcoded neutrals
   - Export with a named export (not default)
   - **Prefer composition over boolean-prop accretion.** A handful of `isX`/`showY`/`hideZ`
     booleans that combine into impossible states is a design smell — accept `children` or
     slot props (e.g. `header`, `actions`) so callers compose what they need.

4. Create a types file in `_lib/` if the component has non-trivial props

5. **Create a co-located `.test.tsx` by default.** Skip it only if the user explicitly opts out.
   The test must assert rendered DOM or behavior — not that a helper returned a value:
   - `@testing-library/react` to render, `@testing-library/user-event` for interactions
   - Assert what the user sees: queried text/roles are present, the right state shows after
     an interaction. A test that renders without asserting anything is not a test.
   - Reuse mocks from `@hadrian-mtv/vitest-utils` where available

## Gotchas

- flow-frontend has no per-package `SKILL.md` indexes. Read package sources (`package.json`
  exports and the `src/` files behind them) — don't assume index files exist.
