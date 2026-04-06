---
name: systematic-debug
description: Structured debugging workflow for diagnosing bugs, test failures, and unexpected behavior. Use when encountering errors, failed tests, build issues, or when user says "debug", "fix this bug", "why is this failing", or "investigate this error".
---

# Systematic Debugging

Find root cause before attempting fixes. Random fixes waste time and create new bugs.

## The Iron Law

No fixes without root cause investigation first. If you haven't completed Phase 1, do not propose fixes.

## Phase 1: Root Cause Investigation

BEFORE attempting ANY fix:

### Step 1: Read Error Messages Carefully
- Read stack traces completely — note file paths, line numbers, error codes
- Check the terminal output from `pnpm lint:tsc`, `pnpm test`, or `pnpm build`
- In this repo, common error sources: TypeScript strict mode violations, ESLint boundary violations, proto type mismatches

### Step 2: Reproduce Consistently
- For test failures: `cd apps/flow-global && TZ=America/Los_Angeles pnpm vitest run path/to/file.test.tsx`
- For type errors: `pnpm lint:tsc`
- For lint errors: `pnpm lint`
- For build failures: `pnpm clean && pnpm build`
- If not reproducible, gather more data — do not guess

### Step 3: Check Recent Changes
- `git diff` for unstaged changes
- `git log --oneline -10` for recent commits
- `git diff HEAD~3` to see what changed recently
- Check if proto types changed (`@bufteam/*` packages) — run `pnpm buf-bump:main` if needed

### Step 4: Trace the Data Flow
- For RPC issues: trace from server action → Connect RPC client → proto types → form state
- For component issues: trace from data source → `rpcToForm*` converter → form state → component props
- For test failures: check if mocks from `@hadrian-mtv/vitest-utils` match current interfaces
- See `references/root-cause-tracing.md` for the full backward tracing technique

## Phase 2: Pattern Analysis

1. **Find working examples** — locate similar working code in the same app or domain
2. **Compare** — what's different between working and broken code?
3. **Check shared packages** — is the issue in `packages/ui-toolkit`, `packages/shared-utils`, or another shared package?
4. **Check domain boundaries** — does the import violate `eslint-plugin-boundaries` rules?

## Phase 3: Hypothesis and Testing

1. **State your hypothesis clearly** — "I think X is the root cause because Y"
2. **Make the smallest possible change** to test it — one variable at a time
3. **Verify** — run the specific failing test or lint check
4. **If it didn't work** — form a NEW hypothesis. Do not add more fixes on top

### After 3+ Failed Fixes: Stop and Reassess
- Each fix revealing new problems in different places = architectural issue
- Ask the user before attempting more fixes
- Consider whether the pattern is fundamentally wrong vs. a surface bug

## Phase 4: Implementation

1. **Run the failing test** to confirm it fails: `cd apps/flow-global && TZ=America/Los_Angeles pnpm vitest run -t "test name"`
2. **Implement a single fix** addressing root cause — one change at a time
3. **Verify the fix**:
   - `TZ=America/Los_Angeles pnpm vitest run path/to/file.test.tsx` — specific test passes
   - `pnpm lint:tsc` — no type errors introduced
   - `pnpm lint` — no lint violations
4. **If fix doesn't work** — return to Phase 1 with new information. Do not retry blindly.

## Common Issues in This Repo

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `Type 'X' is not assignable to type 'Y'` in proto types | Proto dependency out of date | `pnpm buf-bump:<service>` |
| `Cannot find module '@hadrian-mtv/...'` | Package not built | `pnpm build` from root |
| ESLint boundary violation | Cross-domain import | Move shared logic to `app/_lib/` or a shared package |
| Hydration mismatch | Server/client rendering difference | Check for `typeof window`, date formatting, browser-only APIs |
| `console.*` lint error | Wrong logger | Use `@hadrian-mtv/flow-logger` instead |
| `next/link` lint error | Wrong navigation component | Use `FlowLink`/`FlowLinkButton` from `@hadrian-mtv/flow-navigation` |
| Test fails with timezone error | Missing TZ env var | Prefix with `TZ=America/Los_Angeles` |
| Stale build artifacts | Turbo cache | `pnpm clean && pnpm build` |

## Red Flags — Stop and Restart Investigation

If you catch yourself:
- Proposing fixes before tracing the data flow
- Adding multiple changes at once
- Saying "let's try this" without a hypothesis
- Skipping test verification after a fix
- On your 3rd+ fix attempt without going back to Phase 1
