# Flow Frontend — Claude Code Tooling for Product Managers

A complete set of Claude Code agents, rules, and skills for contributing to the flow-frontend monorepo. Designed so Product Managers can plan, design, and build UI features with Claude Code handling the conventions.

## Prerequisites

- Access to the `flow-frontend` repository
- pnpm (>= 10), Node.js (>= 24)
- Claude Code installed

## Install

Individual skills can be installed via the `skills` CLI (skills only — rules and agents require the full bundle below):

```bash
# Install all flow-frontend skills at once
npx skills@latest add mark2685/agent-tools/skills/flow-frontend

# Or install individually
npx skills@latest add mark2685/agent-tools/skills/flow-frontend/new-component
npx skills@latest add mark2685/agent-tools/skills/flow-frontend/new-page
```

To install the full bundle (rules + agents + skills), clone and copy:

```bash
cd /path/to/flow-frontend
git clone --depth 1 https://github.com/mark2685/agent-tools.git /tmp/agent-tools
mkdir -p .claude/rules .claude/agents .claude/skills
cp -r /tmp/agent-tools/rules/flow-frontend/* .claude/rules/
cp -r /tmp/agent-tools/agents/flow-frontend/* .claude/agents/
cp -r /tmp/agent-tools/skills/flow-frontend/* .claude/skills/
rm -rf /tmp/agent-tools
```

Hooks must be merged manually into `.claude/settings.json` (see [Hooks](#hooks) below).

## What Gets Installed

### Rules (load automatically — no action needed)

Rules enforce conventions so you don't have to remember them. All but one are glob-scoped: they load only when files matching their `globs` frontmatter are in play (e.g. `react-patterns.md` activates on `.tsx` edits, `dependencies.md` on `package.json`). The exception is `reviewer-reporting-conventions.md`, which has no globs and applies to every conversation.

| Rule | What it enforces |
|------|-----------------|
| `react-patterns.md` | Derive state inline (memoize only as an escape hatch), stable keys, composition over boolean props, concurrent features |
| `accessibility.md` | WCAG 2.2 AA — semantic HTML, focus appearance, 24x24 targets, no nested interactives, disabled-control reasons |
| `forms.md` | React Hook Form + Zod, pending state from `formState.isSubmitting`/mutation, proto data flow, error surfacing |
| `protobuf.md` | Proto-ES v1 conventions, int64-as-string, RFC 3339 timestamps, proto3-default mapping, generated-file boundaries |
| `nextjs-app-router.md` | Server/client components, server-side fetching (no useEffect fetching), error handling, route organization |
| `shared-packages.md` | Which shared packages to check before writing new code |
| `tanstack-query.md` | `queryOptions()` factories, hierarchical query keys, `onSuccess`/`onError`, no N+1 fetching — the target standard for new code |
| `typescript-conventions.md` | No `as` casts, named exports, single-export files, `null` over `UNSPECIFIED` |
| `testing.md` | Vitest + RTL: assert rendered DOM, real UI path, combinatorial cases |
| `dependencies.md` | No blind dep-update merges, pnpm `catalog:` (strict) alignment, prettier-vs-eslint separation |
| `reviewer-reporting-conventions.md` | Shared Critical/Important/Minor severity vocabulary + human-verification gate for all reviewers |
| `playwright-test-quality.md` | The E2E quality bar — assert visible DOM, real UI path, canaries, combinatorial coverage |

### Agents (invoke when you need a focused workflow)

Agents are specialized Claude Code personas with restricted tool access. Invoke them by name in your prompt.

| Agent | When to use | What it produces |
|-------|------------|-----------------|
| `ux-planner` | "Plan the UX for feature X" | UX Design Brief + Component Layout Plan |
| `design-reviewer` | "Review this design" | Rubric scores, violation flags, recommendations |
| `ui-builder` | "Build feature X" | Implemented code following all conventions |

**Developer-oriented agents** (useful but not required for PMs):

| Agent | Purpose |
|-------|---------|
| `domain-boundary-reviewer` | Checks architecture boundary compliance |
| `rpc-convention-reviewer` | Validates protobuf/RPC patterns |
| `react-quality-reviewer` | Reviews React/TS for correctness, reuse, type tightness, sound abstractions |
| `frontend-test-reviewer` | Reviews Vitest/RTL and Playwright test quality (real assertions, real UI path, canaries) |

**Requires Playwright MCP plugin**:

| Agent | Purpose |
|-------|---------|
| `playwright-test-planner` | Plans E2E test scenarios |
| `playwright-test-generator` | Generates Playwright tests |
| `playwright-test-healer` | Fixes broken E2E tests |

### Skills (invoke via slash command)

Skills are mechanical tasks triggered with a `/` command.

| Skill | Command | What it does |
|-------|---------|-------------|
| `new-component` | `/new-component` | Scaffolds a React component with conventions |
| `new-page` | `/new-page` | Scaffolds a Next.js page with co-located files |
| `buf-bump` | `/buf-bump` | Updates protobuf dependencies (user-confirmed; see [Hooks](#hooks)) |
| `systematic-debug` | `/systematic-debug` | Structured debugging workflow |
| `frontend-testing` | `/frontend-testing` | Scaffolds/strengthens Vitest + RTL tests, hands E2E to Playwright |
| `add-server-action` | `/add-server-action` | Wires a server action to an RPC in `_lib/actions` with proto conversion at the edge |
| `proto-migration` | `/proto-migration` | Fixes converter/server-action/form breakage after a proto bump |
| `e2e-test-fixture` | `/e2e-test-fixture` | Makes Playwright tests runnable against the local stack (seed, auth, port) |
| `receiving-code-review` | `/receiving-code-review` | Triages PR review comments, verifying each is real and the fix landed |
| `brainstorming` | `/brainstorming` | Shapes a rough idea into an approved spec before any code |

## The PM Workflow

### New feature (full flow)

```
1. ux-planner        → Produces a design brief + component plan
2. design-reviewer   → Reviews and refines the brief
3. ui-builder        → Implements the feature (rules enforce conventions)
4. /new-component or /new-page as needed during step 3
```

**Example prompts:**

1. *"Plan the UX for a new supplier approval page in flow-global"* — triggers `ux-planner`
2. *"Review this design brief"* — triggers `design-reviewer`
3. *"Build the supplier approval page based on the design brief above"* — triggers `ui-builder`

### Small changes (skip the agents)

For bug fixes, component tweaks, or adding a field — just describe what you want. The relevant rules load automatically as soon as matching files are touched, and Claude follows the conventions.

**Example:** *"Add a 'notes' field to the order details form in flow-global"*

## Hooks

The `hooks/flow-frontend/settings.json` file contains recommended hooks:

- **PreToolUse (Edit|Write)**: Blocks edits to generated files (`*/generated/*`, `*/lib/openapi/*`, `*/examples/gen/*`), lock files, and CI config — those must not be edited manually — and to env files, which hold user-managed local config and secrets (the agent is told to ask you to edit them yourself). Exception: `.env.development` is editable, because the `e2e-test-fixture` skill and the workspace docs instruct setting local-auth values (e.g. `OKTA_ISSUER`) there.
- **PreToolUse (Skill)**: Appends one `{skill, ts}` JSON line per skill invocation to `.claude/skill-usage.jsonl`, so under-triggered skills can be spotted. Fail-open — any failure still exits 0, so logging never blocks a tool call. Add `.claude/skill-usage.jsonl` to the project's `.gitignore`.
- **PostToolUse (Edit|Write)**: Auto-formats edited files with Prettier — but only files inside the project (`$CLAUDE_PROJECT_DIR`), so edits to sibling repos in the multi-repo workspace are never reformatted under flow-frontend's Prettier config.

To install hooks, merge the contents into your project's `.claude/settings.json`.

The same file also enables three Claude Code marketplace plugins via `enabledPlugins`: `frontend-design` (Figma/visual design inspection used by `design-reviewer` and `ui-builder`), `playwright` (the `playwright-test` MCP server the Playwright agents require), and `typescript-lsp` (TypeScript language-server intellisense). Verify these names against your Claude Code plugin directory before enabling them.

**Dependency:** these hooks require [`jq`](https://jqlang.github.io/jq/). Claude Code passes hook input as JSON on stdin, so the hooks parse it with `jq`. Both Edit|Write hooks precheck for `jq` and fail with a distinct `HOOK ERROR` naming it if missing; the same goes for a missing stdin path or an unset `$CLAUDE_PROJECT_DIR` — they fail loudly on stderr rather than silently skipping the guard. The skill-usage logger is the one exception: it skips silently on any failure, because logging must never block a tool call.

## Keeping in Sync

The source of truth for these assets is this `agent-tools` repository. `flow-frontend/.claude/` is an install target, not the canonical copy: re-run the git clone + copy commands above to pull the latest versions into it. Edits made directly in a project's `.claude/` do not flow back here — make changes in `agent-tools` and re-copy.

## Contributing Back

Discoveries made while working in flow-frontend sessions — a gotcha, a stale fact, a wrong path — should be upstreamed to `agent-tools` via a small PR. Appending an entry to the relevant skill's Gotchas section is the default contribution type. Do not let fixes die in the local `.claude/` copy: it gets overwritten on the next re-install.
