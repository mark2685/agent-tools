# Reviewer Reporting Conventions

The shared contract for every reviewer agent (rpc-convention-reviewer, domain-boundary-reviewer, design-reviewer, react-quality-reviewer, frontend-test-reviewer). Reviewers reference this rule instead of restating it.

## Severity vocabulary

Classify every finding as exactly one of three levels. Not everything is Critical — mis-classifying a nitpick as Critical erodes trust in the whole review.

- **Critical (must fix)** — bugs, security issues, data-loss or data-corruption risks, broken functionality. A silently wrong value reaching the backend (int64 precision loss, a proto3 default written as a real choice) is Critical.
- **Important (should fix)** — architecture problems, missing functionality, poor error handling, test gaps, convention breaks that will cause bugs or rework.
- **Minor (nice to have)** — code style, naming, optimization opportunities, documentation polish.

## How to report a finding

- Be specific: cite `file:line`, not a vague area.
- Explain why it matters, not just what to change.
- Give a concrete fix or the exact existing implementation to reuse.
- Acknowledge what was done well before listing issues — accurate praise helps the author trust the rest.
- Give a clear verdict; do not hedge.

## Human-in-the-loop verification

Reviewer findings are advisory. The human owns the decision and the change.

- Never apply a fix on confidence alone. Surface the finding; let the human confirm it is real and apply it.
- Do not claim something is "fixed" — a reviewer that wrote no code cannot assert a fix landed. Report what you observed and what you recommend.
- When you flag a regression or a suspected silent failure, say what evidence would confirm it, so the human can verify rather than trust.
