# agent-tools — 2026 Best-Practice Review, Pass 2 (Independent)

**Status: REPORT-ONLY.** This is a second, independent best-practice audit of the `2026-best-practice-pass` branch (`f06a092` + `9c581d3`, 2 commits ahead of `main`). It modifies no bundle asset. It was conducted blind — the prior review at `docs/2026-review.md` was not read until all Phase-A findings were recorded — and only then reconciled against it (§4). Findings are advisory; a human owns each decision.

**Resolution addendum (2026-06-11):**

- The **P0 hook exit-code defect** and §7's P1/P2 items 2–12 were **fixed in commit `797230e`** (item 13 was optional and declined; item 14 is outside this repo, as §7 itself notes).
- §6's "staged, not committed" claim about the Anthropic guide **self-invalidated**: this report and the guide landed together in commit `f848e68`.
- `docs/2026-review.md`, referenced throughout, is an **intentionally untracked local working document** (excluded via `.git/info/exclude`), so its references will not resolve in a clone.
- A subsequent fix pass (this branch) addressed the adversarially-verified findings of a third review — see `git log`.

**Reviewer model:** dual-axis — (1) frontend correctness (React 19 / Next 16 App Router / TanStack Query v5 / RHF + Zod v3 / WCAG 2.2 / ProtoJSON / Turborepo) and (2) Claude Code authoring mechanics (skills / agents / rules / hooks).

**Method note:** A 17-agent workflow built the 2026 rubric from primary sources, then audited each changed asset against it and against the live `flow-frontend` checkout. I cross-checked every load-bearing fact myself and **rejected three subagent findings as false positives or weak** (recorded in §3.1) — this report is my own adjudicated verdict, not a pass-through of the agents' output.

---

## 1. Executive summary & headline verdict

> **Verdict: the branch is VALID and substantially conforms to 2026 best practice.** It correctly implemented the prior review's backlog. One residual **P0** remains (the generated-file hook does not actually block), plus a short list of **P1** accuracy/consistency issues and **P2** polish/currency items. None of the P1/P2 items block a merge; the P0 should be fixed before anyone relies on the hook as a guardrail.

The shipped state is good. Of 36 substantive changed assets, **30 are conformant** on both axes as audited. Reviewer agents are correctly read-only (verified tool allowlists); skills observe progressive disclosure and the size budget; `buf-bump` carries `disable-model-invocation: true`; rules declare appropriate `globs`; descriptions are third-person with literal triggers; and nearly every concrete factual claim (package names, exports, scripts, versions, fixture details) checks out against the live repo. The obsidian assets were removed, which is the right call for a shared team repo.

The residual issues are concentrated:

1. **P0 — the central deterministic guardrail is still a no-op, for a new reason.** The prior review caught (and the branch fixed) the `$CLAUDE_FILE_PATH`/`$PROJECT_ROOT` variable bug; the hook now parses `.tool_input.file_path` from stdin JSON and uses `$CLAUDE_PROJECT_DIR` — both correct. **But the PreToolUse generated-file guard exits with code `1`, which Claude Code treats as a *non-blocking* error. Only `exit 2` blocks a tool call.** So the hook prints `BLOCK: …` to stderr and then lets the `Edit`/`Write` proceed. The block never fires. Ironically, the hook's *own* error branches use the correct `exit 2`.

2. **P1 — a WCAG mislabel inherited from the prior review.** `accessibility.md` and `design-reviewer.md` file **SC 2.4.13 Focus Appearance** under "WCAG 2.2 **AA**" with its 3:1-contrast requirement. SC 2.4.13 is **Level AAA**, not AA. The 3:1 figure is correctly attributed to 2.4.13 — it's the *conformance level* that's wrong. This error originated in the prior review's rubric (RU-07) and propagated into the shipped assets.

3. **P1 — the README's sync model is inverted.** "The source of truth … is `flow-frontend/.claude/`" is false as shipped: that mirror has 6 rules / 4 skills / 8 agents and is missing every asset this branch added. The branch (`agent-tools`) is the authoritative copy.

4. **P1/P2 — gating consistency.** `proto-migration` edits source after a bump but, unlike `buf-bump`, is not gated with `disable-model-invocation: true`. Worth a decision (it is less destructive than `buf-bump`, so this is a consistency judgment, not a clear defect).

5. **P2 — polish & currency.** `buf-bump`'s description contains `<service>` (angle brackets are forbidden in frontmatter per Anthropic's guide); `new-component`/`new-page` use imperative ("Scaffold") rather than third-person descriptions; the `"Next.js 15+"` heading is cosmetically stale; `protobuf.md`'s `z.coerce.bigint()` recommendation could note its `TypeError`-on-bad-input edge; and `react-patterns.md`/`nextjs-app-router.md` don't yet mention forward-looking Next 16 / React 19 features (Cache Components, ref-as-prop) the repo hasn't adopted.

The **Next 15→16 / React 19 version drift** the prior review flagged is **resolved in the branch's own assets** — every bundle asset now says "Next 16 / React 19." The only stale "Next.js 15 / Node ≥ 23 / TypeScript 5.9+" text lives in the workspace-level `/Users/mark.richter/Projects/CLAUDE.md`, which is outside this repo and outside the branch's scope (§5).

---

## 2. Methodology & primary sources

### 2.1 What was audited

Change set from `git diff --stat main...HEAD`: 39 files (+1,167 / −517). Substantive assets: **12 rules** (6 modified, 6 new), **10 agents** (8 modified, 2 new), **11 skill files** (5 modified incl. 1 reference, 6 new), the **hooks `settings.json`**, the **bundle README**; plus **4 removed obsidian assets** (3 skills + 1 rule).

Every concrete claim was verified against the live checkout at `/Users/mark.richter/Projects/flow-frontend` (`package.json`, `pnpm-workspace.yaml`, `pnpm-lock.yaml`, `eslint.config.mjs`, package sources). Unverifiable claims are labelled as such rather than asserted.

### 2.2 The 2026 rubric (primary sources)

**Axis 1 — Frontend correctness**

| Area | Standard held to | Primary source |
|---|---|---|
| React 19 | Derive state in render; memoize only as an escape hatch when the Compiler is **not** enabled (it isn't, here); `use()` for context/promises; `useTransition`/`useDeferredValue` by profiling; `cache()` for per-request dedup; stable keys; no components-in-components; immutable updates; ref-as-prop (forwardRef deprecated) | react.dev — [react-compiler-1](https://react.dev/blog/2025/10/07/react-compiler-1), [use](https://react.dev/reference/react/use), [useTransition](https://react.dev/reference/react/useTransition), [useDeferredValue](https://react.dev/reference/react/useDeferredValue), [cache](https://react.dev/reference/react/cache), [react-19-upgrade-guide](https://react.dev/blog/2024/04/25/react-19-upgrade-guide), [you-might-not-need-an-effect](https://react.dev/learn/you-might-not-need-an-effect) |
| Next 16 App Router | Async `params`/`searchParams`/`cookies()`/`headers()`; Server Components by default, `'use client'` at the leaf; no `useEffect` fetching; `error.tsx`/`loading.tsx`; **Cache Components** (`use cache`, `cacheLife()`) is the new caching model in 16 | nextjs.org — [caching/Cache Components](https://nextjs.org/docs/app/getting-started/caching), [upgrading to v16](https://nextjs.org/docs/app/guides/upgrading/version-16), [server-and-client-components](https://nextjs.org/docs/app/getting-started/server-and-client-components) |
| TanStack Query v5 | `queryOptions()` factories; hierarchical/tuple keys; thin `mutationFn` + `onSuccess`/`onError`; `useSuspenseQuery`; prefetch/`ensureQueryData` | tanstack.com — [query-keys](https://tanstack.com/query/v5/docs/framework/react/guides/query-keys), [prefetching](https://tanstack.com/query/v5/docs/framework/react/guides/prefetching) |
| RHF + Zod (v3) | `zodResolver`, `z.infer`, `useFieldArray`, `formState.isSubmitting`; `z.coerce.bigint()` exists in v3 but throws `TypeError` (not `ZodError`) on non-numeric input | react-hook-form.com; zod.dev; [zod#1856](https://github.com/colinhacks/zod/discussions/1856) |
| WCAG 2.2 | New SC and **their exact levels**: 2.4.11 Focus Not Obscured Min (**AA**), 2.4.13 Focus Appearance (**AAA**), 2.5.7 Dragging (**AA**), 2.5.8 Target Size Min (**AA**), 3.3.7 Redundant Entry (**A**) | w3.org — [WCAG22](https://www.w3.org/TR/WCAG22/), [#focus-appearance](https://www.w3.org/TR/WCAG22/#focus-appearance), [target-size-minimum](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum), [dragging-movements](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements), [redundant-entry](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry) |
| ProtoJSON / protobuf-es v1 | 64-bit ints as JSON **strings** (precision past 2^53); `Timestamp` ↔ RFC 3339; proto3 default/absence; v1 uses `bigint`, `Timestamp` class, `PlainMessage<T>` | [protobuf.dev/json](https://protobuf.dev/programming-guides/json/); github.com/bufbuild/protobuf-es |
| Turborepo / pnpm | pnpm `catalog:` single-version + `catalogMode: strict`; turbo `--filter`; deep imports over barrels; knip; prettier vs eslint separation | turbo.build/repo/docs; pnpm.io |

**Axis 2 — Authoring mechanics**

| Area | Standard held to | Primary source |
|---|---|---|
| Skills | `SKILL.md` + frontmatter (`name`, `description`); description third-person with WHAT + WHEN + literal triggers, ≤1024 chars, **no `<`/`>`**; progressive disclosure, SKILL.md ≤ ~500 lines / 5,000 words, references one level deep; `disable-model-invocation: true` for side-effecting skills; `allowed-tools` to restrict | platform.claude.com agent-skills best-practices; code.claude.com [skills](https://code.claude.com/docs/en/skills); **Anthropic, _The Complete Guide to Building Skills for Claude_** (now at `docs/references/anthropic-skill-building-guide.md`) |
| Subagents | frontmatter (`name`, `description`, `tools`, `color`); least-privilege tools; reviewer/researcher agents read-only (no `Edit`/`Write`/`Bash`); description drives delegation | code.claude.com [sub-agents](https://code.claude.com/docs/en/sub-agents) |
| Hooks | input is JSON on **stdin**; `$CLAUDE_PROJECT_DIR` for project root; **PreToolUse: `exit 2` blocks, `exit 0` proceeds through normal flow, any other code (incl. `1`) is a non-blocking error** | code.claude.com [hooks](https://code.claude.com/docs/en/hooks) |
| Rules | path-scoped rules declare `globs`; always-on rules omit them | code.claude.com docs; repo `AGENTS.md` |
| Ecosystem (corroborating only) | kebab-case dirs, frontmatter discipline, trigger-style descriptions, references discipline | addyosmani/agent-skills, vercel-labs/agent-skills, vercel-labs/skills, mattpocock/skills, skills.sh |

### 2.3 Ground-truth versions (live `flow-frontend`, via catalog + lockfile)

| Package | Live | Bundle / workspace claim |
|---|---|---|
| `next` | **16.2.6** | bundle: "Next 16" ✓ · workspace `CLAUDE.md`: "Next.js 15" ✗ |
| `react` / `react-dom` | **19.2.6** | "React 19" ✓ |
| `@tanstack/react-query` | **5.90.20** | "v5" ✓ |
| `react-hook-form` | **7.71.1** | — |
| `zod` | **3.25.76** | v3 (not v4) — affects `z.coerce.bigint` guidance |
| `@bufbuild/protobuf` | **~1.10.1** | protobuf-es **v1** — `protobuf.md` "v1" ✓ |
| `turbo` | **2.9.14** | — |
| `typescript` | **6.0.3** | workspace `CLAUDE.md`: "5.9+" ✗ |
| `tailwindcss` | **^4.3.0** | "Tailwind v4" ✓ |
| node engines | **>=24** | README: "Node ≥ 24" ✓ · workspace `CLAUDE.md`: "Node ≥ 23" ✗ |
| pnpm | 10.30.2 | — |

---

## 3. Per-asset findings

Verdict key: **conformant** (meets the 2026 bar on both audited axes) · **improve** (sound, with a real gap worth a decision) · **problem** (a defect that defeats the asset's purpose). Severity P0 > P1 > P2. Axis: F = frontend, A = authoring.

### 3.1 Findings I rejected or downgraded (independent scrutiny of the audit)

These were raised by the audit agents and I do **not** carry them forward; recorded so a human doesn't chase them:

| Claim | Disposition | Why |
|---|---|---|
| `shared-packages.md` "lacks YAML frontmatter delimiters" | **Rejected (false positive)** | The file opens with `---\nglobs:\n  - "apps/**/*.tsx"\n  - "apps/**/*.ts"\n---`. Delimiters are present and correct. |
| `typescript-conventions.md` uses "universal globs" instead of omitting them | **Rejected** | `**/*.ts` / `**/*.tsx` is correct *TS-file* scoping, not "all files" — it stops the rule from loading when editing `.md`/`.json`/`.css`. Appropriate as written. |
| `reviewer-reporting-conventions.md` "has no frontmatter" → should add `globs: []` | **Downgraded to optional** | Per the repo's own `AGENTS.md` ("no globs = applies to all files"), a frontmatter-less rule is a valid always-on rule — which is the intent here. An explicit signal is a nicety, not a requirement. |
| playwright agents declare the MCP dependency "informally in prose" → want a `requires-mcp` field | **Downgraded** | The prose declaration ("requires the `playwright-test` MCP server … stop and report") satisfies the dependency-declaration bar. No `requires-mcp` frontmatter field exists in Claude Code today, so this is speculative. |

### 3.2 Rules

| Asset | Verdict | Axis | Finding | Severity | Citation |
|---|---|---|---|---|---|
| `react-patterns.md` | conformant | F/A | Compiler-not-enabled framing, memoize-as-escape-hatch, stable keys, derive-inline, concurrent features all current and accurate. *Optional currency:* note React 19 ref-as-prop / forwardRef deprecation (repo still uses forwardRef). | P2 | react.dev/react-19-upgrade-guide |
| `nextjs-app-router.md` | improve | F | `"## Async Patterns (Next.js 15+)"` heading is cosmetically stale (body is Next-16-correct). Does not mention Next 16 **Cache Components** (`use cache`, `cacheLife()`) — *forward-looking only:* the live config does **not** enable `cacheComponents`, so the rule correctly documents current behavior. | P2 | nextjs.org/.../version-16 |
| `tanstack-query.md` | conformant | F/A | `queryOptions()` factories, hierarchical keys, `onSuccess`/`onError`, no-N+1 — matches v5 guidance. Globs scoped to apps. | — | tanstack.com query-keys |
| `forms.md` | conformant | F/A | RHF+Zod accurate. The "this codebase does not use React 19 `<form action>`/`useActionState`/`useFormStatus`" claim is **verified true** (zero usages in apps/packages) — so *not* recommending those is correct, not a gap. | — | react-hook-form.com |
| `protobuf.md` | conformant | F/A | int64-as-string, `Timestamp`↔RFC 3339, proto3-default mapping all correct; protobuf-es **v1** framing matches `@bufbuild/protobuf ~1.10.1`. *Minor:* `z.coerce.bigint()` (line 42) throws `TypeError` (not catchable `ZodError`) on non-numeric input — worth a one-line caveat. | P2 | protobuf.dev/json; zod#1856 |
| `typescript-conventions.md` | conformant | F/A | No-`as`, named/single-export, `null` over `UNSPECIFIED` — sound. Globs correctly scoped to TS files. | — | — |
| `accessibility.md` | improve | F/A | **SC 2.4.13 Focus Appearance is mis-leveled as AA — it is AAA.** Filed under the "WCAG 2.2 AA" heading with the 3:1 requirement, which is a 2.4.13 (AAA) requirement, not an AA one (the AA focus criteria are 2.4.7 Focus Visible and 2.4.11 Focus Not Obscured). *Also:* SC 2.5.7/2.5.8/3.3.7 lack level labels (AA/AA/A). | **P1** | [w3.org/TR/WCAG22/#focus-appearance](https://www.w3.org/TR/WCAG22/#focus-appearance) |
| `dependencies.md` | conformant | F/A | `catalogMode: strict`, `knip`, `eslint-config-prettier` all verified present. Accurate. | — | pnpm.io; verified |
| `shared-packages.md` | conformant | F/A | `cnMerge`, `usePersistedForm`, `useToast`, `theme.css` exports verified; "read the dir, don't maintain a list" is good anti-drift authoring. (Frontmatter delimiters present — see §3.1.) | — | verified |
| `reviewer-reporting-conventions.md` | conformant | A | Shared Critical/Important/Minor vocabulary + HITL gate; valid always-on rule (no globs). | — | — |
| `testing.md` | conformant | F/A | Vitest+RTL: assert DOM, real UI path, mock at the RPC edge, `it.each`, canaries. Glob `**/*.test.ts(x)` correct. | — | testing-library docs |
| `playwright-test-quality.md` | conformant | F/A | Web-first assertions, real UI path, no `test.skip`-to-green, no `waitForTimeout`. Glob `**/*.spec.ts` correct. | — | playwright.dev |

### 3.3 Agents

| Asset | Verdict | Axis | Finding | Severity |
|---|---|---|---|---|
| `react-quality-reviewer.md` (new) | conformant | A | Read-only (`Read, Glob, Grep, LS`); third-person + triggers; clean out-of-scope cross-refs. |
| `frontend-test-reviewer.md` (new) | conformant | A | Read-only; third-person + triggers; complements (doesn't duplicate) the playwright agents. |
| `design-reviewer.md` | improve | F/A | Same **SC 2.4.13 AA→AAA mislevel** as `accessibility.md` (line ~63 under "WCAG 2.2 AA"). Radius tokens "sm 4 / md 6 / lg 8 px" **verified accurate** against `theme.css`. Read-only. | **P1** |
| `domain-boundary-reviewer.md` | conformant | F/A | Read-only. Its domain summary table is simplified (both apps' configs actually declare the same 9 element-types; the live `from: 'orders'` allow-list omits `rootLib` which the table lists) — but the agent **explicitly instructs reading the live config as source of truth and "the eslint config wins,"** which mitigates the drift. Could trim the table. |
| `rpc-convention-reviewer.md` | conformant | F/A | Read-only; int64/Timestamp/proto3 checks accurate; `<form action>` note verified true. |
| `ui-builder.md` | conformant | F/A | `Read, Write, Edit, Glob, Grep, Bash` is justified (it builds). Verify steps (`lint:tsc`, `test`, `build:typesafe-url`) are real scripts. |
| `ux-planner.md` | conformant | F/A | Read-only; correctly defers the rubric to `design-reviewer` (single source). |
| `playwright-test-planner.md` | conformant | A | Read-only; MCP dependency declared; real-UI-path + assertion bar. |
| `playwright-test-generator.md` | conformant | A | No `Write`/`Edit` (writes via MCP tool); quality bar present; example uses correct `async ({ page }) =>` signature. |
| `playwright-test-healer.md` | conformant | A | `Edit` + MCP tools justified for healing; "fix to reflect correct behavior, surface suspected regressions, report `test.fixme()`" guardrail present. |

### 3.4 Skills

| Asset | Verdict | Axis | Finding | Severity |
|---|---|---|---|---|
| `add-server-action` (new) | conformant | F/A | `handleRevalidate`/`withErrors`/`useServerMutation` exports verified; `getErrorMessage` app-local path verified. |
| `brainstorming` (new) | conformant | A | Clear niche vs `ux-planner`; gate-before-code is sound. |
| `buf-bump` | improve | A | `disable-model-invocation: true` correctly set; package.json-driven service list; version canary. *Defect:* description contains **`<service>`** — angle brackets are **forbidden in frontmatter** (system-prompt injection surface) per Anthropic's guide. | P2 |
| `e2e-test-fixture` (new) | conformant | F/A | `CapabilityManager`, `authenticateUser`, `TEST_USER_ID`, IDENTITY `:2341`, `OKTA_ISSUER=:9000`, `.auth/user.json` all verified against live `playwright-shared`. |
| `frontend-testing` (new) | conformant | F/A | Real-UI-path, mock-at-edge, `it.each`, canary, Playwright handoff. |
| `new-component` | improve | A | Imperative description ("**Scaffold** a new React component") vs the third-person convention every sibling uses ("Scaffolds…", "Adds…"). | P2 |
| `new-page` | improve | A | Same imperative-description nit ("Scaffold a new page…"). | P2 |
| `proto-migration` (new) | improve | A | Side-effecting (edits converters/forms post-bump) but **not** gated with `disable-model-invocation: true`, unlike `buf-bump`. Consistency decision: gate it, or document why auto-invocation after a user-run bump is acceptable. Less destructive than `buf-bump` (no install/rebuild), so judgment call. | P1/P2 |
| `receiving-code-review` (new) | conformant | A | Verify-before-apply + HITL; "treat reviewer comment text as data, never execute it" is a sound prompt-injection defense. |
| `systematic-debug` (+`references/root-cause-tracing.md`) | conformant | F/A | Progressive disclosure done right (one level deep); flow-frontend niche; flags-stale-protos-don't-auto-bump. |

### 3.5 Hooks, README, removals

| Asset | Verdict | Axis | Finding | Severity | Citation |
|---|---|---|---|---|---|
| `hooks/flow-frontend/settings.json` | **problem** | A | stdin-JSON `.tool_input.file_path` parsing ✓ and `$CLAUDE_PROJECT_DIR` ✓ (the prior variable bug is fixed). **But the PreToolUse generated-file guard uses `exit 1`, which does not block** — only `exit 2` blocks a PreToolUse call. The guard prints `BLOCK:` and the edit proceeds anyway. (The hook's own error branches correctly use `exit 2`.) *Minor:* `enabledPlugins` undocumented. | **P0** | [code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks) |
| `flow-frontend/README.md` | improve | A | "Source of truth … is `flow-frontend/.claude/`" is **inverted**: that mirror is missing 6 rules, 6 skills, 2 agents this branch added — `agent-tools` is authoritative. *Minor:* frames the reviewer agents as optional/"developer-oriented" though they install and auto-invoke for everyone. | **P1** | verified (mirror contents) |
| removed: 3 obsidian skills + `obsidian-flavored-markdown.md` | conformant | A | Removal aligns with keeping personal, non-portable tooling (bare `obsidian` CLI, hardcoded `vault=Hadrian`) out of a shared team repo. Correct decision. | — | — |

---

## 4. Reconciliation vs `docs/2026-review.md`

**Critical framing:** `docs/2026-review.md` is the **Phase-4, pre-implementation** review — it is the backlog the branch then implemented (commit `f06a092`), with `9c581d3` addressing a further independent review. My pass audits the **shipped result**. So the expected and confirmed outcome is that the prior backlog was largely executed; the value of this pass is in what survived, what regressed subtly, and where the prior review was itself wrong.

### 4.1 Agreements (both passes align)

- **Hooks must actually work as a deterministic guardrail** (prior TM-16; my P0). Both treat a silently-non-functional hook as the worst failure mode.
- **Reviewer agents read-only; least-privilege tools; progressive disclosure; `disable-model-invocation` on side-effecting skills; `globs` scoping; third-person trigger descriptions.** The branch implemented these and my pass confirms them in the shipped state.
- **WCAG 2.2 criteria, protobuf field-type mapping, TanStack v5 key factories, type-tightening, testing rigor** all now have authoring homes — exactly the gaps the prior review's §6 proposed, now filled (`tanstack-query.md`, `typescript-conventions.md`, `testing.md`, `reviewer-reporting-conventions.md`, `playwright-test-quality.md`; skills `frontend-testing`, `add-server-action`, `proto-migration`, `e2e-test-fixture`; agents `react-quality-reviewer`, `frontend-test-reviewer`). I independently judge these conformant.
- **`buf-bump` gating** (prior P0) is implemented and I confirm it.

### 4.2 Contradictions (we disagree — and who is right)

- **SC 2.4.13 conformance level. The prior review is wrong; this pass is right.** The prior rubric (RU-07) and backlog (§4c, P0 item 3) instruct adding "SC 2.4.13 Focus Appearance (always-visible focus at ≥3:1)" under **WCAG 2.2 AA**. Per [w3.org/TR/WCAG22/#focus-appearance](https://www.w3.org/TR/WCAG22/#focus-appearance), **SC 2.4.13 is Level AAA.** The AA focus criteria are 2.4.7 (Focus Visible) and 2.4.11 (Focus Not Obscured, Minimum); neither mandates the 3:1/2px-thick indicator. The branch faithfully implemented the prior review's instruction and therefore **inherited the error** in `accessibility.md` and `design-reviewer.md`. This is the one place the prior review actively introduced a defect.

### 4.3 Misses (real issues the prior review did not catch)

- **The hook `exit 1` vs `exit 2` bug (P0).** The prior review correctly diagnosed the *variable* bug (`$CLAUDE_FILE_PATH`/`$PROJECT_ROOT`) and its recommended fix ("parse from stdin; use `$CLAUDE_PROJECT_DIR`") was applied correctly. But the prior review never examined exit-code semantics, so its fix said nothing about them — and the implementation used `exit 1` for the block. The guardrail is still a no-op. This is the headline new finding.
- **README "source of truth is `flow-frontend/.claude/`" inversion (P1).** Not flagged previously; false as shipped.
- **`proto-migration` gating consistency (P1/P2).** The prior review proposed creating `proto-migration` (§6) but its spec didn't address `disable-model-invocation`; the shipped skill omits it while its sibling `buf-bump` gates.
- **`buf-bump` description contains `<service>` (P2).** Forbidden-in-frontmatter angle brackets; missed by both the prior review and my own audit agents — caught on direct inspection against the guide.
- **Forward-looking currency (P2):** Next 16 Cache Components and React 19 ref-as-prop — both absent from the (Next-15-framed) prior review.

### 4.4 Stale / incorrect claims in the prior review

- **The SC 2.4.13 = AA mislabel** (above) — incorrect, and it propagated into shipped assets.
- **"Next.js 15" rubric framing** (prior §2 RU header, and "every rule says Next.js 15" in Open Question #4) — the prior review's *bonus finding* correctly identified the live repo as Next 16.2.6 / React 19.2.6 and recommended bumping the rules; the branch did so. So the prior review both contained the stale framing **and** flagged it; the branch resolved it. Net: resolved (§5).
- **Did the `9c581d3` hooks fix land correctly?** Partially. Verified against [the official hooks docs](https://code.claude.com/docs/en/hooks): the stdin-JSON `.tool_input.file_path` parsing and `$CLAUDE_PROJECT_DIR` usage are **correct** — the prior review's core diagnosis was right and was fixed. The exit-code defect (§4.3) is what remains.

---

## 5. The Next 15→16 / React 19 version-drift finding (with live-repo evidence)

**Conclusion: resolved within the branch; residual drift is out of scope.**

- **Live evidence** (`pnpm-workspace.yaml` catalog confirmed by `pnpm-lock.yaml`): `next 16.2.6`, `react`/`react-dom 19.2.6`, `typescript 6.0.3`, node `engines >=24`, `zod 3.25.76`.
- **Branch assets are correct:** `react-patterns.md` ("React 19, Next 16"), `nextjs-app-router.md` ("Next 16"), `design-reviewer.md`/`ux-planner.md`/`ui-builder.md` ("Next.js 16 / React 19"), README ("Node ≥ 24"). A repo-wide search for stale strings (`Next.js 15`, `Node >= 23`, `5.9`) returns **only** the cosmetic heading `"## Async Patterns (Next.js 15+)"` in `nextjs-app-router.md` (where "15+" reads as "15 and later" and the body is 16-correct).
- **Residual drift is external:** the workspace-level `/Users/mark.richter/Projects/CLAUDE.md` still says "Next.js 15", "Node >= 23", "TypeScript 5.9+". That file is **not in the `agent-tools` repo** and **not in this branch's change set**, so it is out of scope here — but it's worth a separate fix so the workspace guidance matches reality (actuals: Next 16.2.6 / Node ≥ 24 / TS 6.0.3).

So the branch **conforms** on versions; the only in-branch action is the one-word heading tidy (P2).

---

## 6. Skills-PDF outcome & recommendation

**Delivered.** Downloaded `The-Complete-Guide-to-Building-Skill-for-Claude.pdf` (HTTP 200, 561,652 bytes, `%PDF-1.4`, 33 pages; SHA-256 `66cbcc06df9271e0bb59986582646f0c4c7c94697554f893f937a5a55cef736d`), transcribed to clean GFM at **`docs/references/anthropic-skill-building-guide.md`** with a provenance header (source URL, retrieval date 2026-06-02, checksum) and a third-party attribution/licensing caveat. The file is **staged, not committed.**

> **Licensing caveat:** the guide is third-party material © Anthropic, reproduced verbatim for internal reference only. It carries no separate license grant; do not redistribute outside this context.

**Proposed commit message** (run by the human):

```
docs: add Anthropic skill-building guide as in-repo reference

Transcribe "The Complete Guide to Building Skills for Claude" (Anthropic,
PDF, retrieved 2026-06-02) to GFM markdown under docs/references/ for
authoring reference. Header records source URL, retrieval date, and
SHA-256; includes a third-party-attribution/licensing caveat. Verbatim
reproduction for internal reference only; not original work of this repo.

Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>
```

**Cross-check of the bundle's authoring conventions against the guide** (guide §Fundamentals, §Planning, References A/B):

| Guide rule | Bundle | 
|---|---|
| `SKILL.md` exact name; kebab-case folder; no `README.md` inside skill folders | ✓ all skills; repo-level `flow-frontend/README.md` is the allowed human README |
| Frontmatter `name` + `description`; description states WHAT + WHEN with trigger phrases; ≤1024 chars | ✓ all skills (longest description 439 chars) |
| **No `<`/`>` in frontmatter** (system-prompt injection surface) | ✗ **`buf-bump` description contains `<service>`** — the only violation (P2) |
| Progressive disclosure; SKILL.md ≤ 5,000 words; references one level deep | ✓ (`systematic-debug` → `references/root-cause-tracing.md`) |
| Third-person description voice (guide examples: "Analyzes…", "Manages…") | mostly ✓; `new-component`/`new-page` use imperative "Scaffold" (P2) |
| `allowed-tools` to restrict tool access | N/A for these skills; the **agents** use the `tools:` allowlist correctly (reviewers read-only) |

Net: the bundle's skills conform to the guide except the `buf-bump` `<service>` angle brackets and the two imperative descriptions — all P2.

---

## 7. Prioritized inconsistencies for discussion

All advisory; the human confirms each is real and owns the fix. One-line fixes given.

### P0 — defeats an asset's stated purpose

1. **`hooks/flow-frontend/settings.json` — PreToolUse block uses `exit 1`.** Change the generated-file guard's `exit 1` to **`exit 2`** so the `Edit`/`Write` is actually blocked (only `exit 2` blocks in PreToolUse). Then verify a `*/generated/*` edit is blocked and a `.tsx` is formatted before relying on it.

### P1 — accuracy / consistency

2. **`accessibility.md` — SC 2.4.13 mis-leveled.** Move 2.4.13 to a clearly-marked AAA note (or reframe the always-visible part under SC 2.4.7/2.4.11 AA and keep the 3:1/2px as a 2.4.13 AAA enhancement); add AA/AA/A labels to 2.5.8/2.5.7/3.3.7.
3. **`design-reviewer.md` — same SC 2.4.13 mislevel.** Apply the same relevelling in its WCAG section.
4. **`flow-frontend/README.md` — sync model inverted.** State that `agent-tools` is the source of truth and `flow-frontend/.claude/` is a (currently stale) install target; re-copying flows agent-tools → `.claude/`, not the reverse.
5. **`proto-migration` — gating consistency.** Decide: add `disable-model-invocation: true` to match `buf-bump`, or document why model-invocation is acceptable for the post-bump fix-up.

### P2 — polish / currency

6. **`buf-bump` description** — replace `<service>` with a bracket-free phrasing (e.g. "pull the latest protos for a service") to satisfy the no-angle-brackets frontmatter rule.
7. **`new-component` / `new-page` descriptions** — change "Scaffold" → "Scaffolds" for third-person consistency.
8. **`nextjs-app-router.md` heading** — `"## Async Patterns (Next.js 15+)"` → `"## Async Patterns (Next 16)"`.
9. **`protobuf.md`** — add a one-line caveat that `z.coerce.bigint()` throws `TypeError` on non-numeric input (validate upstream / prefer a string-pattern guard).
10. **`react-patterns.md`** — optional: note React 19 ref-as-prop (forwardRef deprecated, migrate incrementally).
11. **`nextjs-app-router.md`** — optional/forward-looking: a short note on Next 16 Cache Components (`use cache`, `cacheLife()`) for when the apps adopt it (not enabled today).
12. **`hooks/settings.json`** — document the three `enabledPlugins` (purpose of `frontend-design`, `playwright`, `typescript-lsp`).
13. **`domain-boundary-reviewer.md`** — optional: trim the summary domain table (both apps share the same 9 element-types; the live `allow` lists differ from the table) — mitigated already by the "read the live config; it wins" instruction.
14. **(out of scope, separate change)** — update the workspace `/Users/mark.richter/Projects/CLAUDE.md` "Next.js 15 / Node ≥ 23 / TypeScript 5.9+" to the live Next 16.2.6 / Node ≥ 24 / TS 6.0.3.

---

## Appendix — verification log (live `flow-frontend`, 2026-06-02)

Confirmed accurate (not re-listed as findings): `mark2685/agent-tools` install path; zero `useActionState`/`useFormStatus`/`<form action>` usage; `getErrorMessage` app-local export; `@hadrian-mtv/connect-server-actions` `/client`+`/server` exports; `cnMerge`/`usePersistedForm`/`useToast`; `playwright-shared` (`authenticateUser`, `CapabilityManager`, `TEST_USER_ID`, `:2341`); `tailwind-config` `theme.css` + radius tokens (sm 4 / md 6 / lg 8 px); `catalogMode: strict`; `knip`; `eslint-config-prettier`; `eslint-plugin-boundaries` + the 9 domain element-types; protobuf-es v1 (`proto3`/`PlainMessage`/`Timestamp` in active use); all referenced root scripts (`lint:tsc`, `lint:prettier`, `build:rpc`, `build:typesafe-url`, `test:e2e:*`, `dev:flow-*`, `buf-bump:*`).

Workflow provenance: 17 agents (9 research + 8 audit), 2.08M subagent tokens, 504 tool calls; raw structured output retained in the run transcript.
