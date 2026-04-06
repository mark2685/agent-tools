---
globs:
  - "apps/**/*.tsx"
  - "apps/**/*.ts"
---

# Shared Packages — Check Before Writing

Before implementing UI components, utilities, or styling patterns, check `packages/` for existing shared functionality. Do not duplicate what already exists.

## UI Components (`@hadrian-mtv/ui-toolkit`)
All shared React components live in `packages/ui-toolkit/src/`. Available components include:
accordion, alert, badge, button, card, checkbox, choicebox, collapsible, command, copy-button, data-table, date-input, dialog, dropdown-menu, file-input, form, hover-card, input, label, link, multi-row-select, navigation-menu, number-circle, popover, radio-group, select, separator, sheet, skeleton, switch, table, tabs, text, textarea, toast, toggle-group

Also provides: `cnMerge` (class merging), `usePersistedForm`, `useToast`

**Always import from `@hadrian-mtv/ui-toolkit/*` — never rebuild primitives in app code.**

## Styling (`@hadrian-mtv/tailwind-config`)
Custom Tailwind config with HSL design tokens in `packages/tailwind-config/src/`. Provides:
- `hadrianUIPreset` with custom color tokens (`surface-*`, `content-*`, `border-*`)
- Prefer design tokens over hardcoded Tailwind colors (`gray-*`, `slate-*`, `zinc-*`, `white`, `black`)

## Class Variants (`@hadrian-mtv/classname-variants`)
Use for component variant styling instead of inline conditionals.

## Other Shared Packages
- `@hadrian-mtv/shared-utils` — array, date, function, map, number, object, string utilities
- `@hadrian-mtv/part-utils` — part/variant/revision utilities
- `@hadrian-mtv/flow-navigation` — `FlowLink`/`FlowLinkButton` (never use `next/link`)
- `@hadrian-mtv/flow-logger` — logging (never use `console.*`)
- `@hadrian-mtv/flow-auth` — auth/session management
- `@hadrian-mtv/flow-capabilities` — authorization checks
- `@hadrian-mtv/connect-server-actions` — RPC in server actions
- `@hadrian-mtv/vitest-utils` — shared test utilities and mocks
- `@hadrian-mtv/middleware-compose` — middleware composition

**When you need a utility function, search `packages/shared-utils/src/` first.** When you need a React component, search `packages/ui-toolkit/src/` first.
