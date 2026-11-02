## Goal

```markdown
Imported from [issue #14](https://github.com/Knatte18/loomyard/issues/14).

Request from a hub operator (NORCE Models, a numerical simulation repository).

Today Drift-Verify is one autonomous agent: it reads the compare log and the Drift-Baseline report, accepts drift it judges intended with `lym drift accept`, fixes the rest, and when unsure stops for the run's parent. In a numerical code base, whether a drift is acceptable is a judgment the operator wants to make: how large a deviation is acceptable, in which quantities, and whether a change in solver output is a fix or a regression. An autonomous accept can bless a regression.

Wanted: a mode, in the hub's drift config (as `discussion_interactive` is for Discussion), where the last drift step is an interactive session with the operator in the task's worktree. The agent presents the drift per test and per quantity, together with the baseline agent's expectation, discusses it with the operator, and records each acceptance or fix the operator decides, before the run goes on. Drift-Baseline stays autonomous.

The hub's drift config shows no such key today (`enabled`, `capture`, `compare`, `baseline_model`, `verify_model`, `verify_max_runs`).
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - parent-directive-pattern-stale-second-interactive-role (class consistency, severity MEDIUM, open in rounds 1)
  - verbstatus-operator-session-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
