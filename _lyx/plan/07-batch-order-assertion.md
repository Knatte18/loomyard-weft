# Card 7 — batches run in the batchifier's order

**Edit:**
- `internal/websterengine/sequence.go`
- `internal/websterengine/sequence_test.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/render.go`
- `internal/websterengine/suspect.go`
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
- `docs/overview.md`

**Create:**
- `plan:internal/websterengine#CheckBatchOrder` -> `func CheckBatchOrder(batches []batcher.Batch) error`

**Intent:**
Webster runs batches in the order the batchifier returns them, which is plan card order, and stops reordering (Decision 12).
`SequenceBatches` reduces to an assertion.

`sequence.go` keeps `deriveEdges` and `refsIntersect` unchanged and replaces `SequenceBatches`, `Cycle`, the Tarjan walk, `orderComponents` and the heap with `CheckBatchOrder(batches)`: it derives the edges and returns an error wrapping a new sentinel, `ErrBatchOrder`, naming every edge from a batch to an earlier one, naming both batch numbers and the card and ref that make the dependency, and `nil` otherwise.
A cycle between batches always holds such an edge, so it is refused by the same rule, and no condensation or cycle warning remains.
The refusal's way forward: fix the plan's card order so every card follows the cards whose targets it uses, then `lyx webster rebaseline --card NN` naming each card you edited, or `lyx webster run --fresh` for a run that has not begun a batch.

What it can skip or let through: nothing new.
Plan-Gate's `uses-later-target` refusal already holds, so the assertion fires only on a plan that bypassed or predates that gate.

Call sites: `Run` and every webstercli verb that today calls `SequenceBatches` (begin-batch, await-batch, record-batch, recover-batch, rebaseline, validate's scoped path) use the batchifier's output as is and refuse on `CheckBatchOrder`'s error before touching state.
`RunResult.Cycles`, the cycle warnings in `finishMasterDone` and the `cycles` key of `lyx webster run`'s envelope go, with its help text.
Comments in `beginbatch.go`, `recoverbatch.go`, `render.go`, `suspect.go` and `runlevel.go` that name `SequenceBatches` or a sequenced order now say execution order is the batchifier's order.

`sequence_test.go` becomes a `CheckBatchOrder` table: an empty list; forward `Uses` edges; two writers of one target in card order; a batch whose card uses a later batch's target, refused naming both batch numbers; a two-batch cycle, refused.
`runlevel_test.go` drops its `Cycles` assertion.

Docs: `doc.go`'s "Execution order is derived, not declared" section becomes one stating that batches run in the batchifier's order and that the order is asserted, not derived.
`refusal-spec.md`'s webster section gains a `batch order: dependency on a later batch` correctness-halt row with the way forward above.
`docs/overview.md`'s webster bullet and the two `loom-plan-spec.md` sentences that describe webster deriving a topological order and SCC condensation now state that webster runs the batches in card order and refuses a backward dependency; the plan-writer guidance itself is not rewritten (`webster-delete-ordering` keeps that).

**ImpactSummary:** `lyx webster run`'s envelope loses its `cycles` key and `RunResult` its `Cycles` field (no caller outside webstercli reads either); a plan with a backward dependency between batches is refused instead of reordered, which Plan-Gate already prevents.
