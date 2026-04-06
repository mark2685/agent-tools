---
name: buf-bump
description: Bump protobuf dependencies for a backend service and fix any resulting type errors
---

# Proto Bump Workflow

Bump protobuf types from one or more backend services and resolve any resulting breakage.

## Available Services

| Shorthand | Script | Backend Service |
|-----------|--------|-----------------|
| main | `pnpm buf-bump:main` | All services (from main branch) |
| ats | `pnpm buf-bump:ats` | asset-tracking-service |
| cs | `pnpm buf-bump:cs` | core-services |
| dip | `pnpm buf-bump:dip` | dip |
| erp | `pnpm buf-bump:erp` | enterprise-resource-planning-service |
| fes | `pnpm buf-bump:fes` | factory-execution |
| ics | `pnpm buf-bump:ics` | inventory-crib-svc |
| part | `pnpm buf-bump:part` | part-service |
| shared | `pnpm buf-bump:shared` | shared |
| qrms | `pnpm buf-bump:qrms` | quality-requirements-management |
| qto | `pnpm buf-bump:qto` | quote-to-order |
| ref | `pnpm buf-bump:ref` | reference-services |
| tls | `pnpm buf-bump:tls` | tool-logistics |
| tms | `pnpm buf-bump:tms` | task-management-svc |
| wms | `pnpm buf-bump:wms` | warehouse-management-service |

## Steps

1. **Ask** which service(s) to bump (or "all" for `pnpm buf-bump:main`)
2. **Run** `pnpm buf-bump:<service>` to update proto dependencies in `pnpm-workspace.yaml`
3. **Install** updated deps with `pnpm install`
4. **Build** proto packages with `pnpm build:rpc`
5. **Type-check** with `pnpm lint:tsc` to find breakage from proto changes
6. **Fix** any type errors in consuming code:
   - Check `rpcToForm*` and `formToRpc*` converter functions for changed field names/types
   - Check server actions that reference changed RPC methods
   - Check any direct proto type usage in components
   - Use `*Required` types from `hadrian_protoc-gen-es-required` for form validation
7. **Test** with `pnpm test` to verify nothing is broken
8. **Report** a summary of what changed and what was fixed
