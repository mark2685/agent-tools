---
name: frontend-testing
description: Scaffolds and strengthens flow-frontend tests with Vitest + React Testing Library, asserting rendered DOM through the real UI path, and hands E2E off to Playwright. Use when the user says "add a test", "write tests for this component", "strengthen these tests", "test this hook", or "/frontend-testing".
---

# /frontend-testing

Write or strengthen tests that assert real behavior, not helper output. Mock-heavy tests that
bypass the UI are near-worthless: the closer a mock sits to the component, the less the test
proves. Conventions live in the `testing` rule (installed at `.claude/rules/testing.md`) —
follow them; this skill is the procedure.

Runs in **flow-frontend**. Tests are co-located `*.test.tsx`/`*.test.ts` files run by Vitest
in jsdom. The React plugin varies: the apps use `@vitejs/plugin-react-swc`, several packages
use plain `@vitejs/plugin-react` — read the target package's `vitest.config.ts`. One test file
per source file. Run a package's suite with `pnpm test`.

## Steps

1. **Identify the unit and its real entry point.** For a component, that is rendering it and
   driving it through user-visible interactions — not calling its internal handlers directly.
   For a hook, render it inside a host component or `renderHook`.

2. **Render and assert the DOM.** Use `render` + `screen` from `@testing-library/react` and
   `userEvent` for interaction. Every test makes at least one positive assertion on rendered
   output that a user could observe:
   - Query by role/label/text (`getByRole`, `findByText`), not test-id, unless the element has
     no accessible handle.
   - Assert visible state changed (`toBeInTheDocument`, `toBeDisabled`, `toHaveTextContent`),
     not that an internal function was called.
   - `await` async UI with `findBy*`/`waitFor`; `cleanup()` in `afterEach`.

3. **Exercise the real UI path.** Click the actual button, type into the actual field, submit
   the actual form. A test that calls `onSubmit(values)` directly proves nothing about whether
   the form wires up. Mock only at true boundaries — the RPC client / server action — never the
   component's own logic or converters.

4. **Cover combinatorial cases with a table.** When behavior varies across inputs (variant ×
   disabled, role × feature-flag, empty/one/many), drive a `it.each` / `describe.each` table so
   the matrix is explicit and readable. Include the edge rows: empty list, single item, error
   state, loading state.

5. **Add a canary for silent failures.** If a test could pass while the feature is mis-wired
   (e.g. a parity test where both sides read the same stub), add a positive assertion that
   fails loudly when the wiring breaks. A green suite must mean the feature works.

6. **Hand E2E to Playwright.** Vitest covers component and unit behavior. Cross-page flows,
   auth, real navigation, and business-intent journeys belong in Playwright against the running
   stack — do not fake them in jsdom. To author those, use the Playwright planner/generator
   agents; to make them runnable, use the `e2e-test-fixture` skill.

7. **Run and report.** `pnpm test` (or scope to the package). Report what is covered, which
   cases the table exercises, and any behavior you deliberately pushed to E2E.

## Anti-patterns to reject

- Asserting a mock was called instead of asserting the rendered result.
- Testing a converter's output by importing the converter — test it through the component that
  uses it, or test the converter directly as a pure function, but never mock it inside a
  component test.
- A snapshot as the only assertion. Snapshots catch nothing intentional; assert specifics.

## Gotchas

- The apps' vitest configs use `@vitejs/plugin-react-swc`, not `@vitejs/plugin-react`. Copying
  a plugin import from a package config into an app config (or vice versa) silently changes the
  transform.
- Keep `cleanup()` in `afterEach`. Several packages (`primary-nav`, `flow-auth`,
  `connect-server-actions`, others) have no global cleanup in their `vitest.setup.ts`, so
  omitting it there leaks DOM between tests. Where a global setup does call it (the apps,
  `ui-toolkit`), the extra call is idempotent.
