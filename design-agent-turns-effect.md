# Measure the effect of the agent-turns project

`agent-turns-effect` · Note · research, tools

## Idea (operator, 2026-11-01)

Measure the effect of project `agent-turns` as a whole, once its members have landed and runs have gone through with them, rather than one member at a time.

## Measure

- Baseline: the numbers in `agent-turns`' body (`tools/tokencount` and `tools/forkaudit` on `warp-visibility`, `interactive-recipe` and `run-verbs-shared`), and `docs/research/review-turn-audit-2026-11-01` for the review roles.
- After: the darn and tapestry runs that landed after the members, as many as there are; darns weigh most, since they run most.
- Darn writers: turns, calls per turn, shell edits against Edit-tool edits and gate-loop turns, with the scripts of `docs/research/turn-audit-2026-10-28`.
- Webster forks: turns per card, calls per turn and peak context, with `tools/forkaudit`; cost with `tools/tokencount`.
- Review roles: calls per turn, fork wake-up turns and removable turns, with the scripts of `docs/research/review-turn-audit-2026-11-01`.
  This includes `plan-reviewer-batching`, which has the fanned Plan reviewer gather its fork pre-read in parallel waves: count its pre-read turns against the baseline.
- Refused turns: the baseline is 6 turns refused by lym's guards across the five heaviest forks; count them again, and count `awk ... > file && mv` edits, the route the shell-edit guard leaves open.
- The report names which members' behaviour shows in the transcripts and states the sample size beside each number.

## Card gate fix loops decide a cold fixer (operator, 2026-11-01)

Before `agent-turns`, a Webster fork fixed its own card gate's failures in its own session, at 150k to 450k tokens of context a turn; one fork spent 69 turns in that loop.
Since then `gate-failure-report` gives one report listing every failure, and `plan-edits-before-gate` tells agents to fix everything a report shows in one message.

- The report counts, per fork, the turns spent between a failing card gate and its next green one, and their tokens and peak context, beside the 69-turn baseline.
- When that loop still costs much, the report recommends a cold, cheap card gate fixer, and the orch puts it to the operator as a note:
  Go spawns a fresh fixer in the fork's worktree with the gate report, the card and the card's diff; it reads the failing source itself, fixes every failure in one go, reruns only the failed packages and then the card's whole gate; when it clears the gate the fork goes on, and when it does not the failure goes back to the fork or up the recovery ladder as today.
  Webster already has such a fixer for the plan-level verify gate (`lym webster fix-verify`, `webster-prefix-verify-fix-cold.md`), which inherits nothing.
