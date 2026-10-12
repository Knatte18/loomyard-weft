# Fixer report — discussion round 3

## Changes

- F1 (BLOCKING), fixed differently in one part: D3's way forward is now one route per layer, each reaching the task worktree's own config, then `lyx loom resume`.
  A loom `run_settings` value is raised with `lyx config loom --layer local --set <key>=<n>` in the task worktree plus `lyx board upsert`; a tracked-layer value with `lyx config loom --set` in the task worktree; a hub default with `lyx config loom --layer default --set`; a darn value with `lyx config darn --set` from the prime.
  Not made: the proposed `lyx config darn --layer local --set …` route.
  `internal/darnengine/config.go` classifies `review_max_bounces` as `configengine.CategoryGuard`, barred from the local layer, and D3 gives darn's hard cap the same layer policy, so that route would be refused.
  D5 and D9 now point at D3's route, Open risks bullet 2 names it, and the `run_settings` acceptance criterion names the local-layer route.
  The support log's rejected alternatives record why the board-only route fails.
- F2 (MEDIUM): D2's Bound now says that no later review segment reads the accepted artifact on Merge-Review and Darn-Review, and that PR-Gate lands the task with no human read while `alignment_pr_approve` is true, its default.
  The stated bounds are the soft-cap window, the `automatic` flag and, on Merge-Review, the carry-over.
  The downstream-review sentence is limited to Discussion, Plan and Webster.
  Open risks bullet 1 now records the residual risk.
- F3 (MEDIUM): acceptance criteria bullets 2, 4, 5, 7 and 8 now carry the qualifier "on a perch without auto-accept, on a replay, and on a `blocking_escalates` round with an open BLOCKING entry".
- F4 (LOW): D3's round-number sentence now says the Bouncer never continues a round numbered H or higher, so such a round is accepted or escalated whatever produced it, a goto onto the segment's review row included.

## Disputed

None.

## Deferred

None.
