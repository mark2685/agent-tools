# Form Conventions

Rules for building forms in this monorepo.

## Stack

- **React Hook Form + Zod** for all forms — use `z.infer<typeof schema>` for form types
- **`usePersistedForm`** from `@hadrian-mtv/ui-toolkit` for forms that should survive navigation/refresh
- **`useFieldArray`** for dynamic repeating sections (line items, addresses, etc.)

## Proto Data Flow

Form state is never raw proto types. Data flows through converter functions:

1. API response → `rpcToForm*()` → form state (plain objects suitable for RHF)
2. Form state → `formToRpc*()` → API request (proto message construction)

Always validate API response data with `require*` functions from `@bufteam/hadrian_<service>.hadrian_protoc-gen-es-required` before passing to converters.

## Mutations

- Mutations go through server actions (`'use server'`) via `@hadrian-mtv/connect-server-actions`
- Invalidate related TanStack Query caches after successful mutations
- Show loading state on the submit button during mutation (disable to prevent double-submit)

## Validation

- Define Zod schemas co-located with the form (in `_lib/` or next to the component)
- Use `zodResolver` from `@hookform/resolvers/zod`
- Prefer field-level validation messages over form-level error summaries
- For fields derived from proto enums, build Zod enums from the proto enum values
