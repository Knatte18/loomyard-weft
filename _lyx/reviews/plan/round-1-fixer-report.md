# Plan-Review round 1 — fixer report

## Fixed

- **F1** (card 6): checked against `internal/burlerengine/engine_test.go`.
  `TestEngine_Run_ShuttleOutcomes` pins each half's passthrough, and `TestEngine_Run_ForkSubagentsSpecWiring` table-drives a cluster row against a plain row, so the premise holds.
  The card's bare `engine_test.go` target is replaced by the glyphs `internal/burlerengine#TestEngine_Run_ShuttleOutcomes` and `internal/burlerengine#TestEngine_Run_ForkSubagentsSpecWiring`.
  The planned new test is gone: the first gains `StartedAt`, `EndedAt` and `Usage` assertions per half, the second a `wantLenses` column, and the surviving-lens rule stays with `TestProfileValidate_ClusterExclude`.
- **F2** (card 8): the `focus_test.go` target and its direct `excludedLenses` test are dropped.
  The `burler_test.go` table at the producer surface gains a row for a missing usage record making the previous round's entries ineligible, and the card states that the table is the one test of the union and eligibility rules.
- **F3** (card 5): `usage_test.go` gains one table-driven `SessionUsage` test whose rows are the four cases.

## Disputed

None.

## Deferred

None.
