---
name: add-server-action
description: Adds a flow-frontend server action wired to an RPC via @hadrian-mtv/connect-server-actions, placed in the _lib/actions folder with proto conversion at the edge and getErrorMessage surfacing. Use when the user says "add a server action", "call this RPC from the server", "wire up a mutation to the backend", or "/add-server-action".
---

# /add-server-action

The canonical pattern for a server action. Server actions are the only place RPC clients are
called from the app; components and forms never touch the client directly. Proto conversion
happens here, at the edge — domain logic and proto shapes stay out of React.

Runs in **flow-frontend**. Two paired files per domain:
`_lib/actions/<domain>.actions.ts` (`'use server'`) and `_lib/actions/<domain>.client.ts`
(`'use client'`). Place them under the nearest feature's `_lib/actions/`, not at app root,
unless the action is genuinely app-wide.

## Steps

1. **Locate the RPC and its client.** Find the generated client in `lib/proto/<service>` (e.g.
   `purchaseOrderClient`) and the `*Required` request/response types in the
   `@bufteam/hadrian_<service>.hadrian_protoc-gen-es-required` package. If the client doesn't
   exist yet, that is a proto/setup gap — surface it, don't hand-roll a fetch.

2. **Write the `.actions.ts` file** (`'use server'` at top). Wrap each RPC with the right
   helper from `@hadrian-mtv/connect-server-actions/server`:
   - `handleRevalidate(client.method, RequiredResponse, RequiredRequest)` for mutations that
     should revalidate paths/tags.
   - `handleServerFetch(...)` for reads invoked from the server.

   ```ts
   'use server';

   import {
     requireUpdateFooResponse,
     requireUpdateFooRequest,
   } from '@bufteam/hadrian_<service>.hadrian_protoc-gen-es-required/<path>/foo_required';
   import { handleRevalidate } from '@hadrian-mtv/connect-server-actions/server';

   import { fooClient } from '@/lib/proto/<service>';

   export const updateFooAction = handleRevalidate(
     fooClient.updateFoo,
     requireUpdateFooResponse,
     requireUpdateFooRequest,
   );
   ```

3. **Convert at the edge, not in the component.** Callers pass and receive plain form/domain
   shapes; map them to/from proto with `formToRpc*` / `rpcToForm*` converters right here. A
   component must never assemble a proto request or read a raw proto response. Handle
   int64-as-string, RFC 3339 timestamps, and proto3 defaults in the converter — see the
   `protobuf` rule (installed at `.claude/rules/protobuf.md`).

4. **Write the `.client.ts` wrapper** (`'use client'`). Re-export each action through
   `withErrors` from `@hadrian-mtv/connect-server-actions/client` so Connect errors serialize
   across the boundary into a result components can read:

   ```ts
   'use client';
   import { withErrors } from '@hadrian-mtv/connect-server-actions/client';
   import * as actions from './foo.actions';

   export const updateFooAction = withErrors(actions.updateFooAction);
   ```

5. **Surface the why.** In the component, drive the action with `useServerMutation` /
   `useServerMutationThenToast` (from `/client`), or TanStack `useMutation`. Pull the failure
   message with `getErrorMessage` (app-local — `apps/*/app/_lib/ErrorMessage/`, not a package export) and show it — never swallow it or print a generic string.
   Keep `onSuccess`/`onError` separate from the call itself; do not stuff side-effects into the
   mutation function.

6. **Verify.** `pnpm lint:tsc` to confirm types line up, then add or update a test
   (`/frontend-testing`) that drives the real UI path and asserts the error message renders on
   failure and success state renders on success.

## Boundaries

- Server actions live in `_lib/actions`; never inline `'use server'` blocks in
  components.
- No cross-cluster fetching from flow-global — keep each app's actions to its own services.
- Name files and the package by domain, not by app.

## Gotchas

- The package scope is `hadrian_<service>` — e.g.
  `@bufteam/hadrian_factory-execution.hadrian_protoc-gen-es-required`. Dropping the `hadrian_`
  prefix produces an unresolvable import that typechecks only after `pnpm install` of a package
  that doesn't exist. Verify against the `pnpm-workspace.yaml` catalog.
