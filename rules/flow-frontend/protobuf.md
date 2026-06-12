---
globs:
  - "apps/**/_lib/**"
  - "apps/**/*.actions.ts"
---

<!-- Known glob gap: apps/flow-global/app/orders/customers/[id]/edit/EditCustomer.tsx defines
converters outside a _lib/ directory. The cleaner fix is moving that file into the adjacent
_lib/ in flow-frontend (co-location convention) rather than widening these globs. -->

# Protobuf-ES v1 Conventions

This project uses protobuf-es v1 with class-based message instances.

## Construction
- `new MessageType({})` to construct messages
- `new MessageType()` for empty messages

## Validation & Required Types
- `require*` functions from `@bufteam/hadrian_<service>.hadrian_protoc-gen-es-required` packages validate and narrow response types
- Import pattern: `import { requireFooResponse } from '@bufteam/hadrian_<service>.hadrian_protoc-gen-es-required/<path>_required'`
- Always validate API response data before passing to components

## Type Patterns
- `PlainMessage<T>` for plain object representations of proto messages
- `Timestamp` class from `@bufbuild/protobuf` for timestamp fields
- Zod schemas align with proto types for form validation

## Data Flow
- RPC responses flow through `rpcToForm*()` converter functions into form state
- Form state flows through `formToRpc*()` converter functions into RPC requests
- Never use proto types directly in form state

## Field-type mapping rules

These are the conversions `rpcToForm*`/`formToRpc*` exist to get right. Handle them in the converter, never in the component.

### 64-bit integers (`int64`, `uint64`, `sint64`, `fixed64`, `sfixed64`)
- protobuf-es types these as `bigint`. ProtoJSON encodes them as **strings** because a JS `number` loses precision past `2^53`.
- `rpcToForm*`: keep the value as `string` (form inputs are strings) or `bigint`. Never widen to `number` — a part count or work-order ID above `2^53` silently corrupts.
- `formToRpc*`: parse the form string to `bigint` (`BigInt(value)`), not `Number(value)`. Validate the Zod schema with `z.coerce.bigint()` or a string-pattern check, not `z.number()`. (In Zod v3 — the pinned version — `z.coerce.bigint()` throws an uncaught `TypeError`, not a `ZodError`, on `undefined`/non-numeric input. Gate it behind a required-string field, or prefer a `z.string().regex(/^\d+$/)` pattern, so bad input surfaces as a validation error rather than a crash.)

### Timestamps (`google.protobuf.Timestamp`)
- The `Timestamp` class from `@bufbuild/protobuf` carries `seconds` (bigint) + `nanos`. ProtoJSON serializes it as an **RFC 3339** string (`2026-06-01T18:30:00Z`).
- `rpcToForm*`: convert to a `Date` with `timestamp.toDate()`, or to the RFC 3339 string for a date input. Do not read `.seconds` arithmetically in components.
- `formToRpc*`: build with `Timestamp.fromDate(date)`. An empty/unset date field maps to an absent timestamp, not the epoch.

### proto3 defaults and absent fields
- proto3 has no presence for scalars: an absent or zero-value field deserializes to its default (`0`, `''`, `false`, enum 0). The wire cannot distinguish "unset" from "set to default."
- `rpcToForm*`: map proto defaults to the empty form value the UI expects — default-`0` enum to `null` (not a sentinel `UNSPECIFIED`), empty string to `''`, default number to a blank input where "not entered" is meaningful.
- `formToRpc*`: map empty form values back to the proto default. For genuine optional fields, use the `optional` keyword's explicit presence (`field !== undefined`) rather than inferring intent from the default value.

## Generated Files
- Proto types come from BSR via `@bufteam/*` packages under `node_modules` — never editable
- Never edit `packages/flow-navigation/src/generated/` (route types — regenerate with `pnpm build:typesafe-url`)
- Never edit the generated OpenAPI clients in `apps/*/lib/openapi/` — the per-service subdirectories are swagger-typescript-api output; only the thin hand-written wrappers at the top level are editable
- `apps/*/lib/proto/` is hand-written client setup (transports, interceptors) — editable, not generated
