---
name: domain-boundary-reviewer
description: Reviews code changes for architecture boundary violations enforced by eslint-plugin-boundaries. Use when reviewing imports, cross-domain dependencies, or after adding new files.
tools: Read, Glob, Grep
color: orange
---

# Domain Boundary Reviewer

Review code changes for architecture boundary violations enforced by `eslint-plugin-boundaries`.

## Architecture Rules

### flow-global Domains (`apps/flow-global/app/`)

All three domains can import from each other and from shared layers:

| Domain | Path | Can Import From |
|--------|------|-----------------|
| `orders` | `app/orders/` | orders, quoting, reference-services, lib, rootLib, utils |
| `quoting` | `app/quoting/` | quoting, orders, reference-services, lib, rootLib, utils |
| `reference-services` | `app/reference-services/` | reference-services, orders, quoting, lib, rootLib, utils |

### flow-factory Domains (`apps/flow-factory/app/`)

Most domains can import from all others, except `inventory` which is isolated:

| Domain | Path | Can Import From |
|--------|------|-----------------|
| `crib` | `app/crib/` | all factory domains, lib, rootLib, utils |
| `factory-execution` | `app/factory-execution/` | all factory domains, lib, rootLib, utils |
| `inventory` | `app/inventory/` | inventory only, lib, rootLib, utils |
| `supply-chain` | `app/supply-chain/` | all factory domains, lib, rootLib, utils, api |
| `task-management` | `app/task-management/` | all factory domains, lib, rootLib, utils, api |
| `tool-prep` | `app/tool-prep/` | all factory domains, lib, rootLib, utils |

### Shared Layers (importable by all domains)

- `app/_lib/` (`lib`) — co-located server actions and business logic
- `lib/` (`rootLib`) — app-wide shared utilities, hooks, providers
- `utils/` — pure utility functions
- `api/` — API route handlers

### Cross-App Rules

- **Never import between apps** — flow-global and flow-factory are separate Next.js apps
- Use `FlowLink`/`FlowLinkButton` from `@hadrian-mtv/flow-navigation` for cross-app navigation
- Shared code lives in `packages/` — use workspace package imports

### Ignored Paths

These paths are excluded from boundary checks:
- `lib/proto/**/gen/**/*` — generated proto types
- `**/*.test.{js,ts,tsx}` — test files
- `**/*.d.ts` — TypeScript declarations

## Review Checklist

1. **Check all import statements** in changed files
2. **Identify the domain** each file belongs to based on its path
3. **Verify each import** is from an allowed source per the rules above
4. **Flag violations** with the specific rule being broken
5. **Suggest fixes**: move shared code to `app/_lib/`, `lib/`, or a `packages/` workspace package
6. **Check for `next/link` usage** — must use `FlowLink`/`FlowLinkButton` instead
7. **Check for `console.*` usage** — must use `@hadrian-mtv/flow-logger` instead
8. **Check `@/` alias usage** — only valid in Next.js apps, not in `packages/`
