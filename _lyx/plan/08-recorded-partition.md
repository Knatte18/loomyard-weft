# Card 8 — webster records the partition once per run

**Create:**
- `internal/websterengine/partition.go`
- `internal/websterengine/partition_test.go`
- `plan:internal/websterengine#PartitionBatch` -> `type PartitionBatch struct`

**Edit:**
- `internal/websterengine/state.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/beginbatch.go`
- `internal/webstercli/awaitbatch.go`
- `internal/webstercli/recordbatch.go`
- `internal/webstercli/recoverbatch.go`
- `internal/webstercli/validate.go`
- `contracts/specs/refusal-spec.md`
- `contracts/specs/webster-spec.md`
- `PATTERN.md`

**Uses:**
- `plan:internal/websterengine#CheckBatchOrder`
- `plan:internal/batcher#Identity`

**Intent:**
The partition is computed once per run and recorded in `state.json`, so sizes read from a tree the batches are changing never regroup cards mid-run (Decisions 10 and 12).

**State.**
`State` gains `Partition []PartitionBatch` (`json:"partition,omitempty"`), in execution order; a `PartitionBatch` holds `Cards []string` (`NN-<slug>` ids), `Profile string` and `Estimate float64` (omitempty).
A state written before the field existed loads with a nil `Partition`.

**`partition.go`.**
`ExecutionBatches(plan, st, active, sizes)` returns the batches every verb runs:
- no state: `active.Batch` over the whole plan (the fresh partition `validate` and first init compute);
- a state with a `Partition`: the recorded batches mapped onto the current plan's cards by id, each batch keeping its recorded `Profile` and `Estimate`;
- a state without one: `batcher.Identity()` over the whole plan, whatever profile is active, so an in-flight run started before this change runs on as it started.
Then `CheckBatchOrder` over the result.
Mapping refuses, naming the cards, when a recorded id is not a card of the plan or a plan card is in no recorded batch: way forward `lyx webster rebaseline --card NN` naming each card you edited, or restore the plan with `lyx webster restore-plan`.
`RecordPartition(st, batches)` writes the batches' ids, profiles and estimates into `st.Partition`.

**Run.**
`Run` computes the batches after its state phase settles instead of before: on a new state (none on disk, or the `--fresh` re-init) it batches with the active batcher and records the partition into that state before its first `SaveState`; otherwise it uses `ExecutionBatches`.
The `zero execution batches` refusal checks the computed batches as today.
`batchIdentity`'s comment states the scheme in force: a batch is numbered and named by its first card.

**Verbs.**
begin-batch, await-batch, record-batch and recover-batch compute their batches from `ExecutionBatches` over the state they load, under the lease where they already hold it; `validate` with no state computes the fresh partition and refuses its order error, and with a state uses `ExecutionBatches`.
`rebaseline` is card 9's.

What it lets through, and its bound: the recorded partition is replaced only by a successful first init (here) or rebaseline (card 9); every other verb only reads it.

**Tests.**
`partition_test.go` (untagged): no state returns the active batcher's grouping; a recorded partition is kept when the size source changes between calls, the case of a file changed between batches, with a cost batcher whose grouping the change would alter; a state without a partition batches with identity while a grouping batcher is active; a recorded id the plan lacks and a plan card outside the partition each refuse naming the card; a backward dependency refuses through `CheckBatchOrder`.
`runlevel_test.go`: a first-init `Run` under a grouping batcher records the partition with each batch's profile name and estimate.

**Docs.**
`doc.go` describes the recorded partition, the legacy-state fallback and that batches run in the recorded order.
`refusal-spec.md`'s webster section gains a `recorded partition does not match the plan` correctness-halt row.
`webster-spec.md` states that the grouping is computed once per run and recorded in `state.json`.
`PATTERN-batcher-registry` in `PATTERN.md` adds that the partition is fixed per run: computed at first init, recorded in `state.json`, replaced only by a rebaseline.

**ImpactSummary:** webster's `Run` and four bracket verbs plus `validate` read batches from `state.json` instead of recomputing them; under identity the batches are the same as before, and states without a partition keep today's behavior.
