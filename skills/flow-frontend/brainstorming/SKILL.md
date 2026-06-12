---
name: brainstorming
description: Shapes a rough idea into an approved written spec before any code, through one-question-at-a-time dialogue — the generalist planning step ux-planner (UI-only) doesn't cover. Use when the user says "let's brainstorm", "help me shape this idea", "I want to plan this before building", "write a spec for", or "/brainstorming".
---

# /brainstorming

Turn a vague idea into a written spec that's been pressure-tested and approved — before writing
code. Planning is the majority of the work on this team; rushing to implementation is where
unexamined assumptions become wasted work. This is the generalist spec-shaping step:
`ux-planner` covers UI/UX planning only; this covers the whole idea — behavior, data, edges,
boundaries, success criteria.

**Gate: no code, no scaffolding, no file changes until the spec is presented and approved.**
This holds even for "simple" tasks. A single utility, a config tweak, a small form — they all
carry assumptions worth surfacing first.

## Steps

1. **Anchor in context.** Read the relevant code, rules, and any existing plan or PR before
   asking anything. Know which app/package, which domain, and what already exists so questions
   are sharp, not generic.

2. **Ask clarifying questions one at a time.** A single focused question, wait for the answer,
   then the next. Drive down each branch of the decision tree before opening another. Cover:
   - The actual problem and who it's for (not the proposed solution).
   - Inputs, outputs, and the data shapes — including proto/RPC boundaries if any.
   - Edge cases, error states, empty/loading/permission-gated states.
   - What's explicitly out of scope.
   - How you'll know it works (the success criteria).

3. **Refine iteratively.** Feed answers back, name the trade-offs, and offer a concrete default
   with one escape hatch rather than a menu of equal options. Surface alternatives you
   considered and why you'd reject them.

4. **Present the design.** A short written spec: problem, approach, data/boundary decisions,
   edge-case handling, scope cuts, success criteria, and open risks. Concrete enough to build
   from, brief enough to read.

5. **Get explicit sign-off, then review-loop.** Ask the user to approve. Capture pushback,
   revise the spec, present again. Iterate until they sign off — don't proceed on a "looks
   fine."

6. **Hand off to implementation.** Once approved, the spec drives the build. If a
   plan-stress-testing skill (e.g. `grill-me`) is installed, offer it before building;
   otherwise proceed to implementation planning directly. Likewise, if an issue-breakdown
   skill is installed, offer it for splitting the spec into trackable work — this bundle
   does not ship one.

## Notes

- The deliverable is the written spec, even when brief. "I'll just build it" skips the gate.
- If the idea is UI-shaped specifically, hand the UX/visual portion to `ux-planner`; keep the
  behavioral and data spec here.
