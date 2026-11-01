# Card 20 — settled-guard-writer

**Edit:**
- `internal/loomcli#settledSegmentGuard` -> `func settledSegmentGuard(producers []shedengine.ProducerDef, reviewsDir string, amendment func() (loomshed.AmendmentRecord, bool, error), history func() ([]shedengine.HistoryEntry, error)) shedengine.GotoGuard`
- `internal/loomcli#segmentWriter` -> `func segmentWriter(producers []shedengine.ProducerDef, segment, bouncer string, history []shedengine.HistoryEntry) string`
- `internal/loomcli#loomCLI.settledGuard`
- `internal/loomcli#TestSettledSegmentGuard`

**Uses:**
- `internal/loomcli#circlingStatusReader`
- `internal/loomcli/doc.go`

**Intent:** Make the settled-verdict refusal of a goto onto a settled Plan-Review row name the plan writer this run ran (D1), so a split run is sent back to `Parts-Plan-Write` and never to a `Plan-Write` its gate would skip.

`segmentWriter` still starts from the first row outside the segment whose `OnDone` names the Bouncer; when that row's latest history entry is `shedengine.OutcomeSkipped` and a row outside the segment names it as its `OnDone`, it names that row instead, and never steps further, so a run that skipped both writers never names a row before them, such as `Discussion-Bouncer`.
On the tapestry recipe that names `Parts-Plan-Write` when `Plan-Write`'s latest entry is Skipped and `Plan-Write` otherwise, so a split run recovered by a goto onto `Plan-Write`, whose latest entry is then not Skipped, names `Plan-Write`; a recipe without the parts rows is unaffected.
`settledSegmentGuard` takes `history`, read once per refusal; a read error is returned, which Shed turns into a refusal, as a settle record's read error is.
`loomCLI.settledGuard` passes the run's status history through `circlingStatusReader` over the run's status file and lock, an absent status reading as no history.

`TestSettledSegmentGuard`'s producer list gains `Parts-Plan-Write` with `OnDone: Plan-Write`, and it gains rows: a settled Plan-Review refuses naming `Parts-Plan-Write` when `Plan-Write`'s latest entry is Skipped, names `Plan-Write` when its latest entry is Done after an earlier Skipped one, names `Plan-Write` with no history, and names `Parts-Plan-Write` when it and `Plan-Write` are both latest-Skipped, never stepping further; a history read error is returned.

**ImpactSummary:** The guard's one production caller and its test pass the run history; refusal text changes only in which writer it names.
