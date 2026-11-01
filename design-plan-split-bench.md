# Study: Parts-Write and Parts-Plan-Write against Plan-Write

`plan-split-bench` · Task · research, loom

## Seen (orch, 2026-11-01)

`Parts-Write` and `Parts-Plan-Write` landed behind the tapestry flag `plan_split`, off by default: `Parts-Write` splits a task into parts with contracts and shared spots, and `Parts-Plan-Write` writes one plan part per fresh session under a joining chair.
Nothing has benched them against `Plan-Write`; note `plan-writer-context` holds the background (the Plan writer is the costliest session, about 83 turns at a mean context of 183k tokens).

## The study

A darn with profile docs writes the plan of the same input twice, once with `Plan-Write` and once with `Parts-Write` then `Parts-Plan-Write`, and compares the two.
- Inputs: two landed tapestry runs' approved decision records and discussions, `warp-visibility` and `run-verbs-shared`, each replayed at the start of Plan on a scratch branch at its own base commit.
- Per plan: cost and wall time of the writing sessions and their forks, turns and context per session, whether the joined plan passes the ordinary `plan` gate first time, its card count, and the cards that read the same files.
- Quality: one round of the hub's Plan-Review per plan, its finding count and severity.
- Cap: 2 inputs times 2 arms, one repeat, the models and efforts the hub's config gives each producer today: 4 plan writings and 4 Plan-Review rounds, no other agent or model run beyond the darn writer.
- The report goes to `docs/research/` with its scripts; it recommends, it decides nothing.
