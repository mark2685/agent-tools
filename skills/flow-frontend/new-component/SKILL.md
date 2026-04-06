---
name: new-component
description: Scaffold a new React component following project conventions. Use when user asks to "create a component", "add a new component", or "/new-component".
---

# /new-component

Scaffold a new React component following project conventions.

## Steps

1. Ask the user:
   - Component name
   - Which app: `flow-factory` or `flow-global` (or a shared package in `packages/`)
   - Feature directory (e.g., `app/orders/_lib/components/`)
   - Whether it needs a test file

2. **Before creating anything**, search `packages/ui-toolkit/src/` for existing components that could be used or extended instead of creating from scratch.

3. Create the component file:
   - Add `'use client'` directive only if it uses hooks, event handlers, or browser APIs
   - Import UI primitives from `@hadrian-mtv/ui-toolkit/*`
   - Use `@hadrian-mtv/classname-variants` for variant styling
   - Use `cnMerge` from `@hadrian-mtv/ui-toolkit` for conditional classes
   - Use Tailwind design tokens (`surface-*`, `content-*`, `border-*`) not hardcoded neutrals
   - Export with a named export (not default)

4. Create a types file in `_lib/` if the component has non-trivial props

5. Create a `.test.tsx` file if requested, with:
   - `@testing-library/react` for rendering
   - `@testing-library/user-event` for interactions
   - Mocks from `@hadrian-mtv/vitest-utils` where available
