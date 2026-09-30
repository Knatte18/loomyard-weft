# Card 4 — batch-card-record

**Edit:**
- `internal/websterengine/state.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/beginbatch_test.go`

**Intent:**
Record, in each batch's own state record, the card set the batch was begun with, so `lyx webster rebaseline` (card 5) can tell a harmless plan edit from one that renumbers or regroups work already begun.

`BatchState` gains `Cards []string` (`cards,omitempty`): one `NN-<slug>` entry per card of the batch, in the batch's card order.
Add an unexported helper `batchCardIDs(b batcher.Batch) []string` in `state.go`.
`BeginBatch` sets it on the record it writes, and `recoverSpawn` sets it on the recovery record, both from the batch being begun.
A record written before the field existed has no `Cards`; card 5 treats it as the single card `NN-<Slug>`, which is what the identity batcher produced.

Extend `beginbatch_test.go` with `TestBeginBatch_RecordsCardSet` asserting `Cards` on the persisted record after a begin and after a re-begin.
The recovery record's `Cards` is asserted in card 11's tests, which already build a recovery fixture.

**ImpactSummary:** One additive omitempty field written by begin-batch and recover-batch; no reader exists until card 5.
