---
globs:
  - "apps/**/*.tsx"
  - "apps/**/*.ts"
  - "packages/**/*.tsx"
  - "packages/**/*.ts"
---

# Shared Packages — Check Before Writing

Before implementing UI components, utilities, or styling patterns, check `packages/` for existing shared functionality. Do not duplicate what already exists.

## Discover Packages from the Source

Read the live workspace — do not rely on a memorized package list. It drifts:

- `pnpm-workspace.yaml` defines the workspace members (`apps/*`, `packages/*`)
- `packages/*/package.json` gives each package's name; its `src/` shows what it provides
- For UI primitives, read `packages/ui-toolkit/src/` and the `exports` map in its `package.json`

**When you need a utility function, search `packages/shared-utils/src/` first.** When you need a React component, search `packages/ui-toolkit/src/` first.

## House Policy

- **Always import from `@hadrian-mtv/ui-toolkit/*` — never rebuild primitives in app code.** Import via deep subpaths (`@hadrian-mtv/ui-toolkit/button`), never a barrel.
- **No UI-library leakage:** consumers must not depend on ui-toolkit's underlying UI library — import the toolkit's own components and props, never the library it wraps. A shared package leaking its UI library is a boundary violation.
- **Logging:** use `@hadrian-mtv/flow-logger`, never `console.*`.

## Styling (`@hadrian-mtv/tailwind-config`)

Tailwind v4 design tokens, shipped as CSS (no JS preset). Apps consume it by importing the theme in their global stylesheet: `@import '@hadrian-mtv/tailwind-config/theme.css'`.

- Use the design tokens (`surface-*`, `content-*`, `border-*`) over hardcoded Tailwind colors (`gray-*`, `slate-*`, `zinc-*`, `white`, `black`)
- Use `@hadrian-mtv/classname-variants` for component variant styling instead of inline conditionals

Covered by the repo `CLAUDE.md`, not restated here: `FlowLink`/`FlowLinkButton` over `next/link`; `cnMerge` for class merging.

## Naming New Packages

Name shared packages for their domain, not the service that happens to own them today — e.g. `machining-ui-toolkit`, not `factory-execution-ui`. The name should survive a backend re-org.
