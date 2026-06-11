---
globs:
  - "**/package.json"
  - "pnpm-workspace.yaml"
---

# Dependency & Version Hygiene

How dependencies are added, upgraded, and kept aligned across the workspace.

## No Blind Dependency-Update Merges
- Never merge an automated dependency-bump PR (Renovate/Dependabot/etc.) without reading the changelog and release notes for the new version.
- Call out breaking changes explicitly in the PR description, and confirm the lockfile diff and `pnpm lint:tsc` still pass.
- A green CI run is not sufficient sign-off for a major version bump — a human reviews the migration notes.

## Align Versions Across the Workspace
- A dependency must resolve to one version across all packages. Shared versions live in the `pnpm-workspace.yaml` `catalog:`, enforced by `catalogMode: strict` — reference catalog entries (`"react": "catalog:"`) instead of pinning a literal version per package.
- When adding a dependency that already exists elsewhere, match the existing catalog version rather than introducing a second one; if it is not yet in the catalog, add it there.
- `knip` flags unused and duplicated dependencies — keep it green. (A dedicated mismatch linter such as `syncpack` is a reasonable future addition, but the strict catalog is the source of truth today.)

## Keep Prettier and ESLint Concerns Separate
- Prettier owns formatting; ESLint owns correctness/quality. Do not add formatting rules to ESLint.
- `eslint-config-prettier` is in place to switch off ESLint's stylistic rules so the two do not fight — keep it, and run formatting through the `lint:prettier` task, not ESLint.
- Do not add an ESLint plugin whose only job duplicates a Prettier rule.
