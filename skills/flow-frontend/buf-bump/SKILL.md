---
name: buf-bump
description: Bumps generated protobuf packages in the flow-frontend monorepo and fixes the resulting TypeScript breakage. Use when the user says "buf bump", "bump protos", "update proto types", "pull the latest protos for a service", or "/buf-bump". User-invoked only — this rewrites package pins and patches consuming code.
disable-model-invocation: true
---

# Proto Bump Workflow

Runs in **flow-frontend**. Updates the `@bufteam/*` proto packages pinned in the
`pnpm-workspace.yaml` catalog and resolves the TypeScript breakage in consuming code. The
packages are prebuilt on the BSR npm registry (`hadrian.buf.dev/gen/npm/v1`) — nothing is
built locally; `pnpm install` fetches them. Side-effecting: it mutates lockfiles, reinstalls,
and edits source. User-invoked only.

## Prerequisites

- Authenticated to the Buf registry: `buf registry login hadrian.buf.dev` (cross-repo requirement).
  Without it, `buf-bump:*` cannot resolve the upstream modules.

## Step 0: Confirm scope

Ask which service(s) to bump. Read the available targets from the root `package.json` —
do not rely on a memorized list. They drift:

```bash
node -e "console.log(Object.keys(require('./package.json').scripts).filter(s=>s.startsWith('buf-bump')).join('\n'))"
```

`buf-bump:main` pulls every service from `main`; `buf-bump:<service>` pulls one.

## Steps

1. **Capture the before-state** so you can prove the bump changed something:
   `git diff --stat pnpm-workspace.yaml > /tmp/buf-before.txt`
2. **Run** `pnpm buf-bump:<service>` (or `pnpm buf-bump:main`).
3. **Canary — verify the bump actually changed versions.** Run `git diff pnpm-workspace.yaml`.
   If it is empty, the pins were already current OR the bump silently no-op'd (bad auth, wrong
   service). Stop and report — do not proceed as if it succeeded.
4. **Install.** The bump script runs `pnpm install` itself; it is what actually fetches the
   prebuilt packages. Canary: `git diff --stat pnpm-lock.yaml` must show the new `@bufteam/*`
   versions. If the lockfile is unchanged, the install pulled nothing — stop and report; do
   not proceed as if it succeeded. (If the script's install step failed, run `pnpm install`
   yourself and re-check.)
5. **Type-check** with `pnpm lint:tsc` to surface breakage.
6. **Fix the breakage**, distinguishing mechanical from non-mechanical:
   - **Mechanical** (renamed field, widened type, new optional field): update the
     `rpcToForm*` / `formToRpc*` converters, server actions referencing the RPC, and direct
     proto usage in components. Use `*Required` types from `hadrian_protoc-gen-es-required`
     for form validation.
   - **Non-mechanical** (a field was **removed**, an enum value dropped, semantics changed):
     **escalate to the user.** A removed field is a backend contract change — surface it; do
     not silently insert a null-check or default that hides lost data.
7. **Test** with `pnpm test`.
8. **Report** the version delta (from step 3), what you fixed mechanically, and any
   non-mechanical changes you escalated.

## Gotchas

- Do not run `pnpm build:rpc` as part of a bump. Its turbo filter (`turbo build --filter
  *-proto`) matches zero workspace packages — the proto packages moved to prebuilt BSR npm
  packages (Nov 2024), so there is nothing to build locally.
