# evals/

Hand-authored eval **fixtures** for the asset bundles in this repo, per the plan in
`docs/eval-framework.md`. Fixtures are content, not build tooling — that keeps the
`AGENTS.md` "pure content repository" boundary intact. **Runners and generated baselines
live in the sibling `agent-tools-evals` repo** (or the installed `skill-creator` plugin);
nothing here executes anything.

Fixtures are namespaced by bundle (`evals/flow-frontend/`) so a second bundle is additive
config, not a refactor.

## Fixture files

### `<bundle>/model.json`

Pins the judge/runner model so regression signal reflects asset changes, not model drift.

```json
{ "judge_model": "claude-opus-4-8", "pinned": "YYYY-MM-DD", "rationale": "..." }
```

### `<bundle>/drift/claims.json` (Tier 3)

Factual claims that assets make about the target project, verified against the target's
HEAD checkout. Top level: `{_schema, verified, claims: [...]}`. Each claim:

```json
{
  "id": "kebab-case-id",
  "asset": "source-repo path of the asset making the claim",
  "claim": "human-readable fact",
  "check": { "type": "...", "target": "...", "pattern": "...", "expect": true }
}
```

Check types (`target` paths are relative to the target repo root):

- `path-exists` — `target` file or directory exists; `expect: true`.
- `grep` — `pattern` (extended regex) matches in `target`; `target` may be a file, a
  directory, or an array of directories (searched recursively, excluding `node_modules`
  and `dist`). `expect: false` makes it an inverse claim — the pattern must NOT match.
  Inverse claims flag *adoption*, not breakage: when one flips, update the asset's framing
  (see the claim text) instead of treating it as a failure.
- `package-script` — `target` is a script name that must exist in the root `package.json`.
- `pkg-version-major` — `target` is a `pnpm-workspace.yaml` catalog key; the runner strips
  the range prefix (`^`/`~`) and compares the major version to `expect` (an integer).
  Never compare patch digits — pure noise.

### `<bundle>/hooks/cases.json` (Tier 5)

Simulated hook invocations against the bundle's `settings.json` (path in the `settings`
field). Top level: `{_schema, settings, verified, verified_note, placeholders, cases}`.
Each case:

```json
{
  "id": "kebab-case-id",
  "hook": "PreToolUse | PostToolUse",
  "matcher": "the settings.json matcher to select the command",
  "stdin": { "tool_input": { "...": "..." } },
  "env": { "VAR": null },
  "expect": { "exit_code": 0, "stderr_contains": "...", "side_effect": "...", "note": "..." }
}
```

- `stdin` is the JSON piped to the hook command; a plain string means raw (non-JSON) bytes.
- Placeholders: `${CLAUDE_PROJECT_DIR}` = the runner's temp project dir; `${OUTSIDE_DIR}` =
  a temp dir outside it. `env` with a `null` value unsets that variable for the case.
- `side_effect`: `none` | `command-invoked` (assert via a stubbed `pnpm` that logs argv —
  real Prettier is not executed; see `command_contains`) | `log-line-appended` (assert
  `log_line_contains` appears in `log_file`, relative to `$CLAUDE_PROJECT_DIR`).
- All recorded exit codes were verified empirically by piping each payload through the
  actual hook command with bash, not derived by reading the script.

### Target state (not yet authored)

`<bundle>/skills/<skill>/triggering.json` (Tier 1), `<bundle>/skills/<skill>/output.json`
(Tier 2), `<bundle>/rules/<rule>/conflict.json` (Tier 4) — formats in
`docs/eval-framework.md` §4.

## Build order (cheapest first, per plan §5)

1. **Tier 0 + Tier 5** — conformance wrapper + hook stdin simulation (`hooks/cases.json`).
   Deterministic; covers the two confirmed defect classes.
2. **Tier 3** — drift claims against the target's HEAD (`drift/claims.json`).
3. **Tier 1** — triggering query sets (~20 per skill; the dominant labor).
4. **Tier 4** — rule glob checks + LLM conflict scan.
5. **Tier 2** — output evals in a throwaway worktree. Last: most expensive, most coupled
   to target-repo churn.
