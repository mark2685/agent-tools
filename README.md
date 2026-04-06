# agent-tools

A curated collection of agentic tooling — skills, agents, rules, and hooks — for AI coding agents. Built primarily for [Claude Code](https://claude.ai/code), with multi-agent support via `AGENTS.md`.

## What's Inside

| Directory | Purpose | Install location |
|-----------|---------|-----------------|
| `skills/` | Slash-command workflows (installable via [`skills` CLI](https://github.com/vercel-labs/skills)) | `.claude/skills/` |
| `agents/` | Specialized Claude Code personas with restricted tool access | `.claude/agents/` |
| `rules/`  | Always-active convention enforcement | `.claude/rules/` |
| `hooks/`  | Pre/post tool-use hook configurations | `.claude/settings.json` |

## Project Bundles

Assets are organized by project under each category (e.g., `skills/flow-frontend/`, `agents/flow-frontend/`). Each bundle has its own README with installation instructions and workflow documentation.

| Bundle | Description | Guide |
|--------|-------------|-------|
| `flow-frontend` | UI development tooling for the Flow Frontend monorepo | [flow-frontend/README.md](flow-frontend/README.md) |

## Installation

### Individual skills via the `skills` CLI

```bash
npx skills@latest add mark2685/agent-tools/skills/<project>/<skill-name>
```

### Full project bundle via git

```bash
cd /path/to/your-project
git clone --depth 1 https://github.com/mark2685/agent-tools.git /tmp/agent-tools
mkdir -p .claude/rules .claude/agents .claude/skills
cp -r /tmp/agent-tools/rules/<project>/* .claude/rules/
cp -r /tmp/agent-tools/agents/<project>/* .claude/agents/
cp -r /tmp/agent-tools/skills/<project>/* .claude/skills/
rm -rf /tmp/agent-tools
```

## File Formats

### Skills (`.claude/skills/<name>/SKILL.md`)

```markdown
---
name: skill-name
description: "What triggers this skill and what it does"
---

Instructions for the agent...
```

### Agents (`.claude/agents/<name>.md`)

```markdown
---
name: agent-name
description: "When to invoke this agent"
tools: Read, Glob, Grep
color: purple
---

Agent instructions...
```

### Rules (`.claude/rules/<name>.md`)

```markdown
---
globs:
  - "**/*.tsx"
---

Rule content (no globs frontmatter = applies to all files)...
```

## License

MIT
