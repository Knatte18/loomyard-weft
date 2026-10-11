# Card 12 — shedrun declares the driver-busy kind and the resume report file

**Create:**
- `internal/shedrun#StartDriverBusyKind`
- `internal/shedrun#ResumeReportFileName`
- `internal/shedrun#ResumeReport`

**Edit:**
- `internal/shedrun#TestParkMarker`

**Uses:**
- `internal/shedrun#ScratchDir`
- `internal/shedrun#StartNotParkedKind`

**Intent:**
Declare, beside `StartNotParkedKind` and `ParkMarker`, the two names loomcli and battencli share for a resume refused on a busy driver pane (decision D8).

`StartDriverBusyKind` is the envelope "kind" of the `lyx loom start` and `lyx loom resume` refusal for a parked driver whose pane would not take the resume line: shuttle held the send on a draft, an active operator or another busy reading.
Its doc comment says it is retryable once the draft is cleared or sent, and that battencli recognises it by this declared value rather than by message text.
`ResumeReport(l, runID)` returns `ResumeReportFileName` under `ScratchDir(l, runID)`, beside the park marker: the file in which the resume branch keeps the stop-report path it picked for the current park, so every rerun against that park types the same resume line.
Both live under `.lyx` through `ScratchDir`, so `PATTERN-lyxdirs-single-declarer` holds.

Test: `TestParkMarker` gains a row asserting `ResumeReport` sits in the same directory as `ParkMarker` under the run's scratch directory and differs from it.

**ImpactSummary:** Two constants and one path constructor added to shedrun; the test row is the only edit.
