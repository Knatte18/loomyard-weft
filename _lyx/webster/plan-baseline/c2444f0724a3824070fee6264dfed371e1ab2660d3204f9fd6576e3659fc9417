# Card 4 — Batcher takes a size source, webster records the partition and runs batches in order

**Create:**
- `internal/websterengine/partition.go`
- `internal/websterengine/partition_test.go`
- `plan:internal/websterengine#PartitionBatch` -> `type PartitionBatch struct`
- `plan:internal/websterengine#CheckBatchOrder` -> `func CheckBatchOrder(batches []batcher.Batch) error`
- `plan:internal/batcher#Identity` -> `func Identity() Batcher`

**Edit:**
- `internal/batcher/batcher.go`
- `internal/batcher/identity.go`
- `internal/batcher/identity_test.go`
- `internal/batcher/registry_test.go`
- `internal/websterengine/sequence.go`
- `internal/websterengine/sequence_test.go`
- `internal/websterengine/state.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/render.go`
- `internal/websterengine/suspect.go`
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/rebaseline_git_test.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/beginbatch.go`
- `internal/webstercli/awaitbatch.go`
- `internal/webstercli/recordbatch.go`
- `internal/webstercli/recoverbatch.go`
- `internal/webstercli/rebaseline.go`
- `internal/webstercli/validate.go`
- `internal/webstercli/run.go`
- `contracts/specs/refusal-spec.md`
- `contracts/specs/loom-plan-spec.md`
- `contracts/specs/webster-spec.md`
- `docs/overview.md`
- `PATTERN.md`

**Uses:**
- `plan:internal/batcher#DiskSizes`
- `plan:internal/batcher#SizeSource`

**Intent:**
The `Batcher` interface takes the plan, the cards and a size source, and each batch carries the profile that formed it and its estimate (Decision 9).
The partition is computed once per run and recorded in `state.json`, so sizes read from a tree the batches are changing never regroup cards mid-run, and webster runs the batches in the order the batchifier returned them, which is plan card order, and stops reordering (Decisions 10 and 12).
`SequenceBatches` reduces to an assertion.
One card holds all of it because every call that forms batches changes once: from the old `Batch(cards)` plus `SequenceBatches` to the recorded partition checked by the assertion, with the new signature.
Identity is still the only batchifier, so under it the batches are the same as before.

**Batcher** (`internal/batcher`).
- `Batcher.Batch(plan *planparser.Plan, cards []planparser.Card, sizes SizeSource) ([]Batch, error)`.
  `cards` is a contiguous run of `plan.Cards` (the whole list, or a rebaseline's tail later); `plan` carries the glyph language the estimator needs.
- `Batch` gains `Profile string`, the `Name()` of the batcher that formed it, and `Estimate float64`, its segment cost, zero for the identity batchifier, which ignores `plan` and `sizes`.
- `Identity()` returns the identity batchifier without a registry lookup; the fallback for a state without a recorded partition needs it whatever profile is active.
- Every caller that computes batches passes `batcher.DiskSizes(<worktree root>)` from the root it is already told: `Run` passes `deps.Geom.WorktreeRoot`, the webstercli verbs `c.geom.WorktreeRoot`.
  A `Batch` error is refused as raw I/O, naming the plan and "re-run the verb" as the way forward (the `raw I/O` row of `contracts/specs/refusal-spec.md`'s webster section).

**Order assertion** (`sequence.go`).
`sequence.go` keeps `deriveEdges` and `refsIntersect` unchanged and replaces `SequenceBatches`, `Cycle`, the Tarjan walk, `orderComponents` and the heap with `CheckBatchOrder(batches)`: it derives the edges and returns an error wrapping a new sentinel, `ErrBatchOrder`, naming every edge from a batch to an earlier one, naming both batch numbers and the card and ref that make the dependency, and `nil` otherwise.
A cycle between batches always holds such an edge, so it is refused by the same rule, and no condensation or cycle warning remains.
The refusal's way forward: fix the plan's card order so every card follows the cards whose targets it uses, then `lyx webster rebaseline --card NN` naming each card you edited, or `lyx webster run --fresh` for a run that has not begun a batch.
`RunResult.Cycles`, the cycle warnings in `finishMasterDone` and the `cycles` key of `lyx webster run`'s envelope go, with its help text.
Comments in `beginbatch.go`, `recoverbatch.go`, `render.go`, `suspect.go` and `runlevel.go` that name `SequenceBatches` now say execution order is the batchifier's order.

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
`Run` computes the batches after its state phase settles instead of before: on a new state (none on disk, or the `--fresh` re-init) it batches with the active batcher, refuses on `CheckBatchOrder`'s error before it saves anything, and records the partition into that state before its first `SaveState`; otherwise it uses `ExecutionBatches`.
The `zero execution batches` refusal checks the computed batches as today.
`batchIdentity`'s comment states the scheme in force: a batch is numbered and named by its first card.

**Verbs.**
begin-batch, await-batch, record-batch and recover-batch compute their batches from `ExecutionBatches` over the state they load, under the lease where they already hold it, and refuse on its error before touching state; `validate` with no state computes the fresh partition and refuses its order error, and with a state uses `ExecutionBatches`.
`rebaseline` takes its batcher and size source as deps here, so the call site is final and card 7 only changes what `Rebaseline` does with them: `RebaselineDeps` replaces `Batches` with `Active batcher.Batcher` and `Sizes batcher.SizeSource`, `webstercli/rebaseline.go` passes `c.batcher` and `batcher.DiskSizes(c.geom.WorktreeRoot)`, and `Rebaseline` forms the batches with `Active.Batch(Plan, Plan.Cards, Sizes)` and refuses on `CheckBatchOrder`'s error, as it compared against the old `Batches` dep before.
`rebaseline_test.go` and `rebaseline_git_test.go` pass the new deps with the identity batcher.
The recorded partition and the tail re-batch are card 7's.

What it can skip or let through, and its bound: the order assertion lets nothing new through, because Plan-Gate's `uses-later-target` refusal already holds, so it fires only on a plan that bypassed or predates that gate, and it refuses where `SequenceBatches` reordered.
The recorded partition is replaced only by a successful first init (here) or rebaseline (card 7); every other verb only reads it.
A state without a partition runs on the identity batchifier, so the fallback only reproduces what that run started under and costs no more than today.

**Tests.**
`identity_test.go` asserts the profile name and zero estimate on each batch; `registry_test.go`'s probe batcher follows the new signature (card 6 deletes the file with the registry it tests).
`sequence_test.go` becomes a `CheckBatchOrder` table: an empty list; forward `Uses` edges; two writers of one target in card order; a batch whose card uses a later batch's target, refused naming both batch numbers; a two-batch cycle, refused.
`partition_test.go` (untagged): no state returns the active batcher's grouping; a recorded partition is kept when the size source changes between calls, the case of a file changed between batches, with a cost batcher whose grouping the change would alter; a state without a partition batches with identity while a grouping batcher is active; a recorded id the plan lacks and a plan card outside the partition each refuse naming the card; a backward dependency refuses with the `ErrBatchOrder` way forward.
`runlevel_test.go` follows the new `Batch` signature in its direct call, drops its `Cycles` assertion, and a first-init `Run` under a grouping batcher records the partition with each batch's profile name and estimate.

**Docs.**
`doc.go`'s "Execution order is derived, not declared" section becomes one stating that batches run in the batchifier's order and that the order is asserted, not derived, and it describes the recorded partition, the legacy-state fallback and that batches run in the recorded order.
`refusal-spec.md`'s webster section gains a `batch order: dependency on a later batch` correctness-halt row with the way forward above and a `recorded partition does not match the plan` correctness-halt row.
`webster-spec.md` states that the grouping is computed once per run and recorded in `state.json`.
`docs/overview.md`'s webster bullet and the two `loom-plan-spec.md` sentences that describe webster deriving a topological order and SCC condensation now state that webster runs the batches in card order and refuses a backward dependency; the plan-writer guidance itself is not rewritten (`webster-delete-ordering` keeps that).
`PATTERN-batcher-registry` in `PATTERN.md` adds that the partition is fixed per run: computed at first init, recorded in `state.json`, replaced only by a rebaseline.

**ImpactSummary:** every `Batcher.Batch` caller (webster's `Run`, `Rebaseline` and the six webstercli verbs) and the in-package test batchers change signature and read batches from the recorded partition instead of recomputing them; `lyx webster run`'s envelope loses its `cycles` key and `RunResult` its `Cycles` field (no caller outside webstercli reads either); a plan with a backward dependency between batches is refused instead of reordered, which Plan-Gate already prevents; under identity the batches are the same as before, and states without a partition keep today's behavior.
