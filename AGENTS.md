# AGENTS.md

This file provides guidance to AI coding agents working in this repository.

## Overview

**agent-tools** is a curated collection of agentic tooling — skills, agents, rules, and hooks — for AI coding agents. Users install individual skills via the [`skills` CLI](https://github.com/vercel-labs/skills), or install full project bundles via git clone + copy.

## Repository Structure

```
skills/           # Skills — slash-command workflows (SKILL.md files)
agents/           # Agents — specialized personas with tool restrictions
rules/            # Rules — always-active convention enforcement
hooks/            # Hooks — pre/post tool-use configurations
flow-frontend/    # Per-project bundle README and onboarding docs
```

Assets are grouped by project (e.g., `skills/flow-frontend/`, `agents/flow-frontend/`). Each project bundle has its own README (e.g., `flow-frontend/README.md`).

## Commands

No build step. All files are hand-authored markdown and YAML.

```bash
# Validate YAML frontmatter in all SKILL.md files
grep -r '^---' skills/ --include='SKILL.md'

# List all skills, agents, rules
find skills/ -name 'SKILL.md' | sort
find agents/ -name '*.md' | sort
find rules/ -name '*.md' | sort
```

## File Formats

**Skills** — `<name>/SKILL.md`:
```yaml
---
name: skill-name
description: "What triggers this skill and what it does"
---
```

**Agents** — `<name>.md`:
```yaml
---
name: agent-name
description: "When to invoke this agent"
tools: Read, Glob, Grep
color: purple
---
```

**Rules** — `<name>.md` with optional globs frontmatter (no globs = applies to all files).

**Hooks** — `settings.json` with `hooks` and `enabledPlugins` configuration.

## Conventions

- **Directory naming**: kebab-case, descriptive of purpose
- **Skill size**: Keep SKILL.md under ~100 lines; split into `references/` if larger
- **Frontmatter**: YAML only; required fields: `name`, `description`
- **No generated files**: Everything is hand-authored

## Maintenance Conventions

- **No hard-coded volatile facts**: lists, names, versions, and inventories are read from the live source at runtime — never snapshotted into asset bodies, because they drift. Model: `buf-bump` reads its service targets from the root `package.json` and says "do not rely on a memorized list"
- **One owner per convention**: every convention lives in exactly one asset; all others reference it by name, never restate it. Owners: `tanstack-query` (query + query-key conventions), `playwright-test-quality` (e2e quality bar incl. the `test.fixme` policy), `reviewer-reporting-conventions` (severity vocabulary), `accessibility` (WCAG criteria and levels), `react-patterns` (memoization policy), `design-reviewer` (UX-defect checklist + design rubric)
- **Cross-references use installed paths**: `.claude/rules/<name>.md`, `.claude/agents/<name>.md` — never source-repo paths like `rules/flow-frontend/<name>.md`, which dangle after install
- **"Append a gotcha" is the default PR type**: skills carry a `## Gotchas` section that accretes entries from real failures — no boilerplate, no invented entries
- **Declare current vs. target**: a convention asset states whether it describes how the code is today or a standard the code is moving toward

## Boundaries

- **Always**: validate that `name` and `description` frontmatter exist in new skills
- **Always**: preserve existing file structure when adding to a project bundle
- **Never**: modify files under a project bundle without understanding the target project's conventions
- **Never**: add build tooling, package.json, or generated files — this is a pure content repository
