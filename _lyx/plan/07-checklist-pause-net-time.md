# Card 7 — checklist-pause-net-time

**Create:**
- `plan:internal/loomcli#readChecklistPaused` -> `func readChecklistPaused(recordPath, slug string, now time.Time) []pauserecord.Interval`

**Edit:**
- `internal/loomcli#checklistInput`
- `internal/loomcli#loomCLI.checklistPoll`
- `internal/loomcli#renderChecklist`
- `internal/loomcli#checklistVisitsOf` -> `func checklistVisitsOf(status shedengine.Status, startedAt string, paused []pauserecord.Interval, now time.Time) (visits map[string]checklistVisits, order []string, history []checklistVisit)`
- `internal/loomcli#TestRenderChecklist`

**Delete:**
- `internal/loomcli#checklistDetailLines`

**Intent:** D5: paused time is not a row's time.

`checklistInput` gains `paused []pauserecord.Interval`, the pause record's coverage of the pair's slug.
`checklistVisitsOf` takes it and sets each visit's time to its wall span less `pauserecord.Excluded(paused, start, end)`, the open visit included, floored at zero; a row's time and a review segment's time (card 5) stay the sums of their visits' net times.
`pauserecord.Excluded` already counts overlapping intervals once and clips them to the span, and a live entry's interval runs from its `PausedAt` (grace included) to now, so an open visit under a live pause stops growing.

`checklistPoll` resolves the record once through `hubgeom.PauseGeometry(location)` (the CLI resolves, the renderer is told) and on every frame calls `readChecklistPaused(geometry.RecordPath, geometry.Slug, now)`, which returns `pauserecord.ReadCoverage`'s `Intervals`.
A geometry that cannot be resolved passes no intervals, and `readChecklistPaused` returns none for a record that cannot be read or decoded, so it counts nothing as paused and never breaks the frame.
A machine sleep with no pause entry, and a pause resumed more than `pauserecord.Retention` ago, are not in the record and so count as row time.

New entries in `TestRenderChecklist`:
- A visit spanning a resumed pause entry of its slug and a hub entry that overlap counts its span less the union of the two, the overlap counted once.
- An open visit under a live pause shows the time up to the pause, the same at two `now` values.
- No intervals (the unreadable-record case) counts the whole span.
- A subtest block inside `TestRenderChecklist` (a `t.Run` beside the table loop, not a table row) calls `readChecklistPaused` over a temporary record file: a resumed entry of the slug and a hub entry give both intervals, an entry of another slug gives none, and a file holding undecodable bytes gives none.

`checklistDetailLines`, which card 4 stopped calling, is deleted.

**ImpactSummary:** Confined to the checklist file and its test; adds one read-only read of the hub pause record per frame and deletes one uncalled helper.
