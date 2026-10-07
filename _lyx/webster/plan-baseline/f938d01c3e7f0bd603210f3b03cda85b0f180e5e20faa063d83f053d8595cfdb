# Card 2 — Tell the batchifier how many batches run before its cards

**Edit:**
- `internal/batcher/batcher.go`
- `internal/batcher/cost.go`
- `internal/batcher/identity.go`
- `internal/batcher/doc.go`
- `internal/batcher/cost_test.go`
- `internal/batcher/identity_test.go`
- `internal/websterengine/partition.go`
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/partition_test.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/runlevel_test.go`

**Intent:**
Implement D5: the batchifier is told the position of the first card run it is handed.

`Batcher.Batch` gains a final `before int` parameter: the number of batches the run executes ahead of `cards[0]`.
The cost batchifier places its k-th returned batch at position `before + k`, so the start context, the feasibility test against `Budget`, the `Estimate` and the Breakdown's `Position` all use that position.
The identity batchifier ignores it.

Callers in `internal/websterengine`: `batchCards` and `formBatches` take `before`; first init (`newPartition` in `Run`, `runlevel.go`), `validate` and `ExecutionBatches` pass 0 (also the identity fallback for a state with no partition); `rebaselineBatches` passes the number of batches it keeps ahead of the regrouped tail.
The partition stays computed once per run and replaced only by a rebaseline (`PATTERN-batcher-registry`).
State the bound in `rebaselineBatches`' comment: the count is derived in Go from the recorded partition, never typed by an agent, and is told as if Merriam were never resumed, so after a resume the estimate is conservative; unmodelled work between batches makes it too low, nothing re-checks it at run time.
`doc.go` names the parameter in the cost-batchifier paragraph.

Test doubles implementing `Batcher` (`sizeBatcher` in `partition_test.go`, `fixedBatcher` in `rebaseline_test.go`, `emptyBatcher` and `failingBatcher` in `runlevel_test.go`) take the new parameter.

Tests:
- `cost_test.go`: a row where cards that fit one batch at `before` 0 split further when handed over at a later `before`, with each Breakdown recording `before` plus its index in the returned list plus one; a card over `Budget` at a late position is card 1's row, which already pins that it runs alone with its over-budget Estimate.
- `identity_test.go`: a non-zero `before` leaves the output unchanged.
- `rebaseline_test.go`: with a real `batcher.NewCost` profile whose `BatchGrowth` is non-zero, the regrouped tail's first batch records position kept-batches + 1 in its Breakdown (a row in `TestRebaseline_CardSet` or the nearest existing rebaseline test).
- `partition_test.go`: `ExecutionBatches` with no state forms the partition from position 1.

**ImpactSummary:** the Batcher interface gains one parameter; every webster batching call site passes a derived count and every Batcher test double follows.
