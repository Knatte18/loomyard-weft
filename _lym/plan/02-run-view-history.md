# Card 2 — run-view-history

**Edit:**
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#ChildReading`
- `internal/battencli#DobbyRuns`
- `internal/battencli#readNoticeChild`
- `internal/battencli#TestDobbyRuns_ReadsTheKeptCancelTheHandOverMarkTheOwnerAndTheBoltHold`
- `internal/battencli#TestReadNoticeChild`

**Intent:**
The run watch (card 3) reads batten's own reason, and the episode stamp of each layer (D1).
The Added after Discussion entry "Episode stamp counts folded repeats" makes that stamp the layer's history length plus the last history entry's `Repeats`.
`shedengine`'s `appendOrFold` folds a repeated budget-exempt Stuck into the previous entry, so the history length alone misses a second block at the same row.
`RunView` carries none of batten's three values, and `ChildReading` lacks the child's `Repeats`.

`RunView` gains three fields beside `State` and `Producer`:
`HistoryLen int`, the length of batten's status history; `LastRepeats int`, the `Repeats` of its last history entry, zero for an empty history; and `Error string`, batten's status reason.
`DobbyRuns` fills them from `run.status.History` and `run.status.Error` in the `RunView` literal it builds for each run.
`ChildReading` gains `LastRepeats int`, the `Repeats` of the child's last history entry, which `readNoticeChild` fills beside `HistoryLen`.
No run notice reads the new field.

Tests:
`TestDobbyRuns_ReadsTheKeptCancelTheHandOverMarkTheOwnerAndTheBoltHold` asserts the three fields for the run it reads.
Its `reading` struct gains the history length, the last entry's repeats and the reason.
The shared batten status constants the test writes for `subject` (`running` and `keptCancel`) gain a known history, whose last entry carries `repeats: 2`, and a reason, so every row's `want` names them.
`TestReadNoticeChild` gains a row whose child status's last history entry carries `repeats: 2` and wants `LastRepeats` 2 beside its `HistoryLen`.

**ImpactSummary:** Three fields on `RunView` and one on `ChildReading`; every existing literal of either in tests stays valid since the fields are zero by default, and `DobbyRuns` and `readNoticeChild` are their one production writers.

**Commit:** 2: Carry batten's history length, repeats and reason in a dobby's run view
