---
globs:
  - "apps/**/*.tsx"
  - "packages/**/*.tsx"
---

# Form Conventions

Rules for building forms in this monorepo.

## Stack

- **React Hook Form + Zod** for all forms — use `z.infer<typeof schema>` for form types
- **`usePersistedForm`** from `@hadrian-mtv/ui-toolkit` for forms that should survive navigation/refresh
- **`useFieldArray`** for dynamic repeating sections (line items, addresses, etc.)

## Proto Data Flow

Form state is never raw proto types: API responses flow through `rpcToForm*()` converters into form state, and form state flows through `formToRpc*()` converters into RPC requests, with `require*` validation at the response edge. Converter and field-type mapping details live in the `protobuf` rule (installed at `.claude/rules/protobuf.md`).

## Mutations

- Mutations go through server actions (`'use server'`) via `@hadrian-mtv/connect-server-actions`
- Reach for `useMutation` (TanStack Query) over hand-managed `isLoading`/`error` state; let it own the async lifecycle
- Invalidate related queries in `onSuccess`; keep success and failure handling in `onSuccess`/`onError`, not stuffed inside `mutationFn`
- Surface the "why" on failure: derive a user-facing message with `getErrorMessage` (app-local — `apps/*/app/_lib/ErrorMessage/`, not a package export) rather than rendering a raw error or a generic string

## Submit State

- Drive submit/pending state from the form/mutation lifecycle — React Hook Form's `formState.isSubmitting` or the mutation's `isPending` — not a hand-rolled `isSubmitting` boolean. (`useActionState`/`useFormStatus` apply only to React 19 `<form action>` server-action forms, which this codebase does not use.)
- Disable the submit control while pending to prevent double-submit
- Render disabled states up front so the layout does not shift — reserve space for inline errors and pending controls so fields do not jump when state changes

## Validation

- Define Zod schemas co-located with the form (in `_lib/` or next to the component)
- Use `zodResolver` from `@hookform/resolvers/zod`
- Prefer field-level validation messages over form-level error summaries
- For fields derived from proto enums, build Zod enums from the proto enum values
