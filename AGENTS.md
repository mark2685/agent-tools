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

## Boundaries

- **Always**: validate that `name` and `description` frontmatter exist in new skills
- **Always**: preserve existing file structure when adding to a project bundle
- **Never**: modify files under a project bundle without understanding the target project's conventions
- **Never**: add build tooling, package.json, or generated files — this is a pure content repository
