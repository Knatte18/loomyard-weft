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
- The report names which members' behaviour shows in the transcripts and states the sample size beside each number.
