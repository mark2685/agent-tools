# Flow Frontend — Claude Code Tooling for Product Managers

A complete set of Claude Code agents, rules, and skills for contributing to the flow-frontend monorepo. Designed so Product Managers can plan, design, and build UI features with Claude Code handling the conventions.

## Prerequisites

- Access to the `flow-frontend` repository
- pnpm (>= 10), Node.js (>= 23)
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

### Rules (always active — no action needed)

Rules apply to every Claude Code conversation automatically. They enforce conventions so you don't have to remember them.

| Rule | What it enforces |
|------|-----------------|
| `react-patterns.md` | Composition over boolean props, compound components, performance patterns |
| `accessibility.md` | WCAG 2.1 AA — semantic HTML, keyboard navigation, ARIA, contrast |
| `forms.md` | React Hook Form + Zod, proto data flow, mutations, validation |
| `protobuf.md` | Proto-ES v1 conventions, `require*` validation, generated file boundaries |
| `nextjs-app-router.md` | Server/client components, data fetching, error handling, route organization |
| `shared-packages.md` | Which shared packages to check before writing new code |

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
| `buf-bump` | `/buf-bump` | Updates protobuf dependencies |
| `systematic-debug` | `/systematic-debug` | Structured debugging workflow |

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

For bug fixes, component tweaks, or adding a field — just describe what you want. The rules are always active and Claude follows the conventions automatically.

**Example:** *"Add a 'notes' field to the order details form in flow-global"*

## Hooks

The `hooks/flow-frontend/settings.json` file contains recommended hooks:

- **PreToolUse**: Blocks edits to generated files (`*/gen/*`, `*/generated/*`, `*/lib/openapi/*`), environment files, lock files, and CI config
- **PostToolUse**: Auto-formats edited files with Prettier

To install hooks, merge the contents into your project's `.claude/settings.json`. The PostToolUse hook uses `$PROJECT_ROOT`, which Claude Code sets automatically at runtime to the project root directory.

## Keeping in Sync

The source of truth for these assets is `flow-frontend/.claude/`. If rules, agents, or skills are updated there, re-run the git clone + copy commands above to get the latest versions.
