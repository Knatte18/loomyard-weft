# Card 17 — implementer-guard

**Create:**
- `internal/loomshed#implementerGuard`
- `internal/loomshed#weaverProducer`
- `internal/loomshed#NewWeaverProducer`
- `internal/loomshed#weaverProducer.Call`
- `internal/loomshed#TestImplementerGuard`

**Edit:**
- `internal/loomshed#websterProducer.Call`
- `internal/loomshed#NewWebsterProducer` -> `func NewWebsterProducer(name string, dirs configengine.Dirs, run shedadapters.WebsterRunner, deps websterengine.RunDeps, commit func() error, active weaverengine.ActiveDeps) shedengine.ShedProducer`
- `internal/shedrecipe#websterEntry`
- `internal/shedrecipe#TestImplementerEntry_WebsterBuildsTheWebsterProducer`
- `internal/loomshed#TestWebsterProducer_Call`
- `internal/loomshed#TestWebsterProducer_CommitsTheRunRecordOnDoneOnly`
- `internal/loomshed#TestWebsterProducer_PassesStuckReasonThrough`
- `internal/loomshed#TestGateRows_StopsNameAWayForward`
- `internal/loomshed#TestBatcherRows_StuckSurfacesTheBatcherError`
- `internal/loomshed#TestCancellation_RealProducersReturnErrorNotStuck`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomshed/doc.go`

**Uses:**
- `internal/weaverengine#Active`
- `internal/websterengine#ArchiveRunAfterReset`
- `internal/weaverengine#ArchiveRunRecord`

**Intent:** The D1 guard, on both implementers' rows, and Weaver's lazy producer (D1, D2).

- `implementerGuard` asks the seam (`weaverengine.Active`) for the run's active implementer.
  When the other implementer has begun batches in the current generation it returns the refusal: Weaver's row names `lym webster reset --to start`, Webster's row names `lym weaver reset`, and both name setting `tapestry.implementer` back with `lym config tapestry --set implementer=<name>`.
  A seam refusal (both begun, or a corrupt state file) is returned as the refusal with its own ways forward.
  A zero `ActiveDeps.AnchorRoot` is a wiring error, returned as an error rather than a refusal.
- `websterProducer.Call` and `weaverProducer.Call` run the guard first and return Stuck with its text, spawning nothing; a run whose other implementer has nothing begun proceeds unchanged.
- `NewWebsterProducer` gains the seam's deps; `websterEntry` passes the ones the recipe `Env` carries (card 18 fills them).
  Every other caller passes an `ActiveDeps` over its test anchor: the listed loomshed tests and the loomrecipe shape table's Webster row.
- `weaverProducer` resolves the active batchifier on every `Call` as `websterProducer` does, with the same Stuck on a `batcher.yaml` fault, and delegates to `shedadapters.NewWeaverProducer`.
  Weaver commits its run record itself (card 12), so it takes no commit seam.
- `loomshed/doc.go` describes the guard on the implementer row.

`TestImplementerGuard` is tier0 over planted state files:
Weaver's row with Webster begun is Stuck naming `lym webster reset --to start` and the setting; after `websterengine.ArchiveRunAfterReset` archives Webster's record, as `reset --to start` does, Weaver's row proceeds;
Webster's row with Weaver begun is Stuck naming `lym weaver reset`; after Weaver's record is archived in place, as `lym weaver reset` leaves it, Webster's row proceeds;
with Webster begun and the setting set back to `webster`, Webster's row proceeds;
a corrupt state of either refuses naming its repair; neither begun proceeds; a zero anchor is a returned error.
The existing Webster producer tests pass the seam's deps over an empty anchor.

**ImpactSummary:** Re-signs one loomshed constructor; its production caller in shedrecipe and every test caller (loomshed tests, the loomrecipe shape table) are listed; Webster's row gains a refusal only when Weaver has begun batches.
