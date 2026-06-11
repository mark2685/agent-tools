# agent-tools — Eval Framework Plan

**Status: PLAN.** This document specifies how we evaluate changes to the assets in this repo — skills, agents, rules, and hooks. It is a design teammates can read and follow; no runner code lives here yet. Decisions below were resolved in a design session and are the contract for any future implementation pass.

---

## 1. What this is (and is not)

A **manual, tiered eval toolkit**, not a CI gate. The tiers are commands you run *before shipping a change*, matched to cost. The cheapest tier is fast enough to be a pre-PR habit; the expensive tiers are tools you reach for when a change warrants them.

This honors the `AGENTS.md` boundary ("no build tooling, no package.json, no generated files — pure content repository") by splitting the framework in two:

- **In this repo:** hand-authored, declarative **fixtures** (labeled query sets, rubrics) and this plan. Fixtures are content, not build tooling.
- **In a sibling `agent-tools-evals` repo (or the installed `skill-creator` plugin):** the **runners** (Python/promptfoo) and **generated baselines**. These are exactly what the boundary forbids in-repo.

Two reusable systems already exist in `skill-creator` and we build on them rather than reinventing:

- **System B — triggering optimizer** (`scripts/run_loop.py`): fully automated. Tests whether a skill's *description* fires on the right queries, with a train/held-out split and auto-refinement picked by held-out score. This is the engine for Tier 1.
- **System A — output benchmark** (`aggregate_benchmark.py` + grader/analyzer prompts): agent-orchestrated with/without-skill grading. We use its aggregation and viewer, but replace its LLM-judged free-text assertions with a deterministic toolchain oracle (Tier 2).

The dynamic-workflow patterns from Anthropic's [harness-for-every-task post](https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code) — fan-out verifier-per-rule, adversarial/blind judging to counter self-preferential bias — are the orchestration layer for the collision pass (Tier 1) and the rules-conflict scan (Tier 4).

---

## 2. The tiers — what each evaluates against

Asset correctness only manifests when a model loads an asset and acts. We therefore evaluate at three levels — static file conformance, triggering behavior, and end-to-end output — plus drift, rules, and hooks.

| Tier | Assets | Criteria (the "evaluate against") | Oracle | Cost |
|---|---|---|---|---|
| **0 · Conformance** | all | frontmatter present; kebab `name` ≤64; `description` ≤1024; **no `<`/`>` in frontmatter**; SKILL.md within line budget; `references/` exactly one level deep; `disable-model-invocation: true` on side-effecting skills only; description = third-person verb + WHAT + WHEN + literal trigger phrases; agent `tools` allowlist matches read-only vs. write intent | mechanical (extend `skill-creator/scripts/quick_validate.py`) | ~1s |
| **1 · Triggering** | skills | ≥90% fire on positive queries, ≤10% false-fire on near-miss negatives, **pass@3, full roster loaded**; collision report on ambiguous queries | `run_loop.py` extended to load siblings; ~20 queries/skill | moderate |
| **2 · Output** | code-producing skills only¹ | produced code passes `tsc`, ESLint + `eslint-plugin-boundaries`, prettier (no diff), build, co-located Vitest; LLM-judge **only** for subjective UI-intent match | live flow-frontend in a throwaway worktree, pass@k | high |
| **3 · Drift** | skills + rules with factual claims | major/minor version claims, export names, script names, and ESLint-rule names still exist (ignore exact patch digits) | flow-frontend **HEAD** | low |
| **4 · Rules** | rules | each rule's `globs` match real files (flag dead or over-broad globs; flag unscoped always-on that should be narrowed); no contradiction vs. another rule or the canonical guide | glob test + LLM conflict scan | moderate |
| **5 · Hooks** | hooks | simulated stdin JSON → correct exit code (2 = block `*/generated/*`, `*/lib/openapi/*`, `*.env*`, `pnpm-lock.yaml`, `*/.github/*`; 0 = allow otherwise); PostToolUse runs prettier on the edited path | dynamic execution | low |

¹ Output evals apply only to skills that emit code: `new-component`, `new-page`, `add-server-action`, `e2e-test-fixture`, `proto-migration`. Reviewer/planner agents and rules do not emit code.

---

## 3. Cross-cutting rules

- **Primary sources win every conflict.** Authoritative: the Anthropic skill-building guide (`docs/references/anthropic-skill-building-guide.md`) for mechanics; primary sources (e.g. w3.org for accessibility) for facts; flow-frontend HEAD for repo facts. The `docs/2026-review*.md` rubric demotes to a **human-review checklist** for non-mechanizable criteria only — it is not an oracle, because pass-2 proved it can carry and propagate a wrong fact (a WCAG criterion mis-leveled AA when it is AAA). (Note: `docs/2026-review.md` is an intentionally untracked local working document, excluded via `.git/info/exclude` — that path will not resolve in a clone; only `docs/2026-review-pass-2.md` is committed.)
- **Pin the model.** Each fixture records the judge/runner model ID (currently `claude-opus-4-8`). Re-baseline deliberately on a model upgrade, so regression signal reflects asset changes, not model drift.
- **Drift is first-class, scoped.** Tier 3 runs against flow-frontend HEAD and flags stale claims, but only for things that matter (major/minor versions, existence of exports/scripts/rules) — never exact patch digits, which would be pure noise.
- **Bundle-namespaced, YAGNI.** Build for flow-frontend only, but namespace fixtures by bundle so a second bundle (e.g. a Go-service bundle) is additive config, not a refactor. No speculative multi-bundle abstraction against a single real example.

---

## 4. Repo layout for fixtures

Fixtures are hand-authored content and live in this repo, kept **out of the installable asset directories** so `skills add` / bundle copies stay clean. The first fixtures now exist under `evals/flow-frontend/` — `drift/claims.json`, `hooks/cases.json`, and `model.json`, authored in the same pass that commits this plan; the rest of the layout below is still target state:

```
evals/
  flow-frontend/
    skills/
      new-component/
        triggering.json     # ~20 labeled queries: {query, should_trigger}
        output.json         # scenarios + deterministic-grader config (code-producing skills only)
      <skill>/...
    rules/
      <rule>/conflict.json  # optional: known-intentional-overlap notes
    drift/claims.json       # asset claims to verify against flow-frontend HEAD
    hooks/cases.json        # simulated stdin JSON → expected exit code / side effect
    model.json              # pinned runner/judge model ID for this bundle
```

Triggering fixture format (matches `skill-creator` System B): a flat JSON array of `{"query": str, "should_trigger": bool}`, ~20 items, 8–10 positives + 8–10 tricky near-miss negatives. Draw negatives from sibling skills/commands so collisions surface.

Runners and generated baselines live in the sibling `agent-tools-evals` repo; nothing generated is committed here.

---

## 5. Build sequence (cheapest-first)

1. **Tier 0 + Tier 5** — wrap `quick_validate.py`; write the hook stdin-simulation harness. Deterministic, days of work, and it covers the two confirmed defect classes (frontmatter angle-brackets; the hook exit-code silent no-op that static review missed in pass-1).
2. **Tier 3 drift** — diff scoped claims against flow-frontend HEAD.
3. **Tier 1 triggering** — author the ~20-query sets (the dominant labor: ~200 queries across the skills), extend `run_loop` to load the full roster and emit a collision report.
4. **Tier 4 rules** — glob-match against the real tree + LLM conflict scan.
5. **Tier 2 output** — worktree harness + deterministic graders. Last because it is the most expensive and most coupled to flow-frontend churn.

---

## 6. Known risks

- **Query-set labor.** Authoring and maintaining ~200 trigger queries is the dominant cost; if it stalls, Tier 1 stalls.
- **Discipline-dependent cadence.** Manual invocation means Tiers 1/2 protect only as far as someone remembers to run them. The cheap tier as a pre-PR habit covers the high-frequency defect classes regardless.
- **Triggering-proxy fidelity.** `skill-creator` tests a description by writing it into `.claude/commands/` as a slash command, not an installed plugin skill. Validate early that this faithfully mirrors how an installed `SKILL.md` competes for routing before trusting Tier 1 numbers.
- **Output-tier coupling.** Worktree runs are slow and flow-frontend conventions move underneath them; expect Tier 2 fixtures to need maintenance as the target evolves.
