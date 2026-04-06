---
globs:
  - "**/proto/**"
  - "**/rpc/**"
  - "**/*rpc*"
  - "**/*proto*"
  - "**/*formToRpc*"
  - "**/*rpcToForm*"
  - "**/*fetcher*"
---

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

## Generated Files
- Never edit files in `lib/proto/*/gen/` or `lib/openapi/` — these are generated
- Proto types come from BSR via `@bufteam/*` packages
