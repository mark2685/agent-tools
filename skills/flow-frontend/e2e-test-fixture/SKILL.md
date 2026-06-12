---
name: e2e-test-fixture
description: Makes flow-frontend Playwright tests runnable against the real stack — seeds employee + capabilities, authenticates through the authtest OIDC flow, and points the runner at flow-factory:3001 or flow-global:3000. Use when the user says "set up an e2e fixture", "my Playwright test won't authenticate", "seed state for e2e", "run e2e against the local stack", or "/e2e-test-fixture".
---

# /e2e-test-fixture

The Playwright planner and generator agents produce tests that assume an authenticated user and
seeded state. This skill supplies that scaffolding so generated tests actually run against the
running stack. Without it they fail at the sign-in redirect.

Runs in **flow-frontend**. E2E lives in `packages/playwright-flow-factory` (targets
`flow-factory` on port 3001) and `packages/playwright-flow-global` (targets `flow-global` on
3000). Shared auth/seeding helpers live in `packages/playwright-shared` (`support/` modules,
re-exported from `index.ts`). Read that `index.ts` for the current fixture API — do not rely
on a memorized list of exports. Names drift.

## Prerequisites

- The target app and its backing services are running locally (see `local-cluster` in the
  workspace `CLAUDE.md`). Identity service must be reachable for seeding.
- The app is configured to authenticate against the local `authtest` mock OIDC:
  set `OKTA_ISSUER=http://localhost:9000` in the app's `.env.development`.

## Steps

1. **Pick the app package.** `playwright-flow-factory` (3001) or `playwright-flow-global`
   (3000). Each has its own `playwright.config.ts` and a `setup` project that runs
   `support/auth.setup.ts` before the test project, saving storage state to `.auth/user.json`.

2. **Seed the user and capabilities** in `auth.setup.ts` using the capability helper exported
   by `@hadrian-mtv/playwright-shared` (it talks to the identity service —
   `IDENTITY_SERVICE_URL`, default `http://localhost:2341`). Read
   `packages/playwright-shared/index.ts` and `support/capabilities.ts` for the current API and
   the shared test user id rather than trusting a remembered signature. Upsert the test
   employee, then grant the minimum capabilities the scenario exercises — broad grants hide
   permission-gating bugs.

3. **Authenticate** via the auth helper from `@hadrian-mtv/playwright-shared`
   (`support/auth.ts`). It walks the NextAuth + authtest redirect chain
   (`/api/auth/signin` → authtest → callback → home), verifies the session, and writes storage
   state to `.auth/user.json`. The test project reuses it via `storageState` so tests start
   logged in.

4. **Seed domain state the scenario needs** before the test acts — create the work order, part,
   or PO through the same backend services, not by stubbing the UI. The test should drive the
   real stack; faked state is the mock-heavy anti-pattern the team rejects.

5. **Point the runner at the stack.** Leaving `PLAYWRIGHT_BASE_URL` unset makes the config
   auto-start `pnpm dev:flow-<app>` on the right port; set it (e.g. in CI/Docker) to run against
   an already-running server and skip the auto-start.

6. **Run.** `pnpm test:e2e:flow-factory` or `pnpm test:e2e:flow-global`. On failure, open the
   report with the matching `:report` script and check the trace.

## Notes

- The shared test user id and the auth/capability helpers live in `playwright-shared` — reuse
  them; do not hardcode a new user id per package.
- If sign-in flakes with a proxy 502, the auth helper already retries; a persistent failure
  means the app isn't up or `OKTA_ISSUER` isn't pointed at authtest.
