# Card 4 — runplan-shared-readers

**Create:**
- `internal/runplan#DoneCards`
- `internal/runplan#EditedDoneCards`
- `internal/runplan#LastBegunCard`
- `internal/runplan#PendingPartition`
- `internal/runplan#RecordBeginPartition`
- `internal/runplan#TestPendingPartition`

**Edit:**
- `internal/websterengine#DoneCards`
- `internal/websterengine#EditedDoneCards`
- `internal/websterengine#LastBegunCard`

**Extends:**
- `internal/runplan/doc.go`

**Uses:**
- `internal/websterengine/doc.go`
- `internal/runplan#State`
- `internal/runplan#RecordPartition`
- `internal/runplan#CardID`
- `internal/runplan#BatchIdentity`
- `internal/websterengine#TestLastBegunCard`
- `internal/websterengine#TestEditedDoneCards`

**Intent:** Move the run-plan readers loom needs onto `runplan.State`, so the active-implementer seam (card 6) serves Webster's and Weaver's state alike (D11, D13), and add Weaver's partition of the remaining cards (D3).

- `runplan.DoneCards`, `EditedDoneCards` and `LastBegunCard` take `*runplan.State` and hold the bodies `websterengine`'s functions of the same names hold today, a nil state answering as now.
  `websterengine`'s three become one-line delegates over `&st.State` (nil-safe), so their callers stay as they are.
  The moved bodies keep their coverage through the delegates: `TestLastBegunCard` and `TestEditedDoneCards` run them unchanged, and `TestNewPlanGate` reaches `DoneCards`; no runplan test repeats them.
- `PendingPartition` drops the plan cards whose card slug a terminal done batch record of `st` holds, matched by slug through each record's `Cards` ids, and keeps those done batches as `st.Partition` records them, in order.
  It partitions the remaining cards, in card order, with `b.Batch`, `before` set to the number of done batches.
  It returns the kept done batches and the fresh batches; a plan with no remaining card returns no fresh batch.
- `RecordBeginPartition` replaces `st.Partition` with the done batches as given followed by the fresh batches recorded as `RecordPartition` records them.
  Weaver calls it at each batch begin, so the later-reference check and `IsLastBatch` see the pending cards and only the run's true last batch reads as last.
- `runplan/doc.go` names the done-card readers and the partition of the remaining cards.

`TestPendingPartition` is tier0, over a fake batcher: a fresh run partitions every card; a done batch's cards are dropped by slug and the batch kept, also after the card was renumbered; a plan whose every card is done returns no fresh batch; `RecordBeginPartition` keeps the done batches first.

**ImpactSummary:** Three websterengine readers become delegates over runplan; callers in loomshed and loomcli compile unchanged.
