---
globs:
  - "**/*.test.ts"
  - "**/*.test.tsx"
---

# Unit / Component Testing — Vitest + RTL

Tests run under Vitest with React Testing Library. Run with `TZ=America/Los_Angeles` (the app `test` scripts set this) so date-dependent assertions match the team's timezone.

## Assert behavior, not implementation

A test is worth only as much as the path it exercises. The closer a mock sits to the UI, the less the test proves.

- Assert what the user sees: query the rendered DOM via accessible roles/text (`getByRole`, `findByText`) and assert on it. Do not assert the return value of a helper, transformer, or formatter that the component itself doesn't call the same way.
- Exercise the real UI path: render the real component and drive it with `@testing-library/user-event` (click, type, submit). Do not call an internal handler directly or reach past the component to invoke an RPC/server action — that bypasses the code you are claiming to test.
- Mock at the network/RPC edge, not at the component or hook boundary. A component that re-implements its rendering inside the test is not under test.

## Cover combinatorial cases

When behavior depends on several inputs (flags, statuses, roles, variants), enumerate the combinations with `it.each` / a table rather than testing one happy path. State the expected output per row so the table is readable and a missing combination is obvious.

## Canaries against silent failure

If a test could pass while the feature is mis-wired (e.g. a parity test where both sides read the same stubbed value), add a canary: an assertion that fails loudly when the wiring breaks. A trivially-passing test is worse than none.

## Mechanics

- One `describe` per unit; name each `it` for the behavior, not the method.
- Prefer `findBy*` / `await waitFor` for async UI over fixed timeouts.
- Co-locate the test next to its subject: `OrderCard.tsx` → `OrderCard.test.tsx`.
- Use `require*`-narrowed fixtures, not `as`-cast plain objects, so a proto shape change fails the test.
