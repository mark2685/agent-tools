---
globs:
  - "**/*.ts"
  - "**/*.tsx"
---

# TypeScript Conventions

## Tighten types instead of casting

An `as` cast is a claim the compiler can no longer check. Almost every `as` is a type that is too loose upstream — fix it there.

- Narrow with a type guard, `switch`, or discriminated union, not `as`.
- Validate external data (RPC responses, JSON) with the `require*` helpers or a Zod schema, then carry the narrowed type — do not cast the raw value.
- `as const` for literal inference and `as unknown` at a genuine FFI boundary are acceptable; a value-to-value `as` to silence an error is not.
- `as any` and non-null `!` are red flags; if a value can be absent, model it as `T | null` and handle it.

## null over sentinel UNSPECIFIED

Use `null` for "no value." Do not pass a proto `*_UNSPECIFIED` enum member around as if it were a real choice. Map `UNSPECIFIED` to `null` at the proto→form edge and back at the form→proto edge; component and form state should never see the sentinel.

## Named exports, single-export files

- Named exports only — no `export default`. Default exports lose their name at the import site and rename silently.
  - **Exception: Next.js App Router special files.** `page.tsx`, `layout.tsx`, `error.tsx`, `loading.tsx`, `not-found.tsx`, `template.tsx`, `default.tsx`, and `global-error.tsx` require a default export — use one there. (`route.ts` is also framework-shaped: it exports named HTTP-method handlers like `GET`/`POST`.) The named-exports rule applies everywhere else.
- One export per file, and the filename matches the export: `getErrorMessage.ts` exports `getErrorMessage`; `OrderCard.tsx` exports `OrderCard`. A single file per export creates a real module boundary.
- No barrel files (`index.ts` that only re-exports) and no junk-drawer `utils.ts` holding unrelated helpers. Import directly from the source module; barrels defeat tree-shaking and blur boundaries.
- Co-locate the type with its consumer or export it from the file that owns it; do not collect unrelated types in a shared `types.ts`.
