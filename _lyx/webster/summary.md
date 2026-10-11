# Burler round as one seat table

Batch 01 (`01-shuttle-hold-deferral`, head `154d4adf5`) added the generic capabilities to shuttle and the seat engine.
Shuttle gained a told hold-notice deferral and a run handle that reads its own session state.
The seat engine gained a per-advisor gate, per-seat fork authorization and the table's round number.
It also gained a required advisor with an acceptance hook that ends the step on any unaccepted end and reports its terminal result, and a chair held-turn notice that waits while an advisor is mid-turn or its up line is pending.
Burler's geometry is now told the shortname and slug its seat strands are named from.
Deviations: the fork dropped the `gated` parameter from `probeSeat` because a seat's gate is now derived from the seat, put `advisorWork` in a new `internal/seatengine/work.go`, and raised the `internal/seatengine` row in `cmd/lyx/testdata/test-budget.yaml`.
The fork also reported using `sed` once for a field rename, which the edit rule forbids; the result is correct.

Batch 07 (`07-retire-fix-start`, head `f525c80b6`) removed `fix_start` and the after-review start order.
A round now runs as one seat table, with the fixer as chair and the reviewer as required advisor-1.
The orchestrators became seat stencils, and the fixer waits with `lyx shuttle await`.
`lyx burler await-review`, the `FixStart` type and its constants, and burlerengine's own Shuttle seam and `RunnerShuttle` were deleted.
Deviation: `seatengine`'s `join` now also stops a chair that timed out, so a timed-out fixer does not stay live and block the next attempt's strand name; the doc, comment and a test row were updated to match.
The fork also dropped the `fix_start` test rows and an explicit-parallel row that duplicated the default.

Both batches recorded `done`.
The forks report passing card gates and the integration batch gate; the `llm`-tier smoke tests were compiled but not run.
The plan-level verify gate runs after this turn.
