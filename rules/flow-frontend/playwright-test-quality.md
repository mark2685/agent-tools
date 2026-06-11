---
globs:
  - "**/*.spec.ts"
---

# Playwright Test Quality Bar

The shared quality standard for end-to-end tests. The playwright-test-planner and playwright-test-generator both reference this rule so a weak test cannot propagate down the pipeline. Specs live in `packages/playwright-flow-factory/tests/` and `packages/playwright-flow-global/tests/`.

## Assert visible DOM and state

Every test ends in at least one positive assertion on what the user can see — `expect(locator).toBeVisible()`, `toHaveText`, `toHaveURL`, a row appearing in a table. A test that only clicks and navigates without asserting an outcome proves nothing. Assert the rendered result, not an internal value.

## Drive the real UI path

Reach state the way a user would: navigate, click, type, submit. Do not seed the screen by calling an RPC, a server action, or `page.evaluate` to inject state that the UI is supposed to produce — that bypasses the flow under test. Locate elements by accessible role / visible text / `getByTestId`, not brittle CSS chains.

## Add canaries against silent pass

A test must fail when the feature breaks. Where a screen could render in a passing-looking state while mis-wired (empty list rendered as "no results", a default value shown as a real one), add a canary assertion that fails loudly on the broken state. Prefer asserting the specific expected content over asserting mere presence.

## Cover combinatorial cases

When behavior varies by input (role, status, flag, variant), parameterize the scenario across the meaningful combinations rather than testing one happy path. State the expected outcome per case so a missing combination is visible.

## Mechanics

- Use the correct async signature: `test('...', async ({ page }) => { ... })`.
- One business intent per spec; name the test for the user outcome, not the click sequence.
- Prefer web-first assertions (auto-retrying `expect(locator)`) over fixed `waitForTimeout`.

## Never disable a test to get green

Never `test.fixme()` / `test.skip()` to get green — a disabled test silently drops coverage; surface the failure instead. The single sanctioned exception is the escalation path of the `playwright-test-healer` agent (`.claude/agents/playwright-test-healer.md`): when a test cannot be healed because the application behavior appears wrong (a suspected regression), the healer may mark it `test.fixme()` only together with (a) a comment stating the actual vs. expected behavior and (b) an explicit report of every fixme'd test back to the user. Silent disabling is never acceptable.
