# Card 9 — Batten's driver-busy notice quotes the box text

**Edit:**
- `internal/battenshed#innerRunProducer.noticeDriverBusy` -> `func (p *innerRunProducer) noticeDriverBusy(ctx context.Context, w *childWait, cause error)`
- `internal/battenshed#innerRunProducer.awaitingStep`
- `internal/battenshed/innerrun_awaiting_test.go`
- `internal/battenshed/doc.go`

**Intent:**
The last clause of D8: batten's driver-busy notice to the orch carries the box text the hold found.
`noticeDriverBusy` takes the spawn error that refused the resume and appends it to the notice's condition, run through `oneLine` and cut at `noticeErrorMax` as `awaitingStep`'s busy report already does, so the quoted box text from the loom refusal reaches the orch.
`awaitingStep` passes its `spawnErr`.
The notice key stays the condition constant, so one stall still sends one notice.

This reverses the current rule that the notice never carries the refusal's draft text: `TestInnerRun_AwaitingRefusedResumeRetriesWithoutMarkerThenResumes` and its doc comment, which pin that rule today, now assert that each queued notice holds the quoted draft of the spawn error the test builds.
`oneLine` already turns control characters into spaces, so the draft reaches the orch pane as one line.
`battenshed/doc.go`'s busy-driver sentence says the notice quotes the box.

**ImpactSummary:** only `awaitingStep` calls `noticeDriverBusy`; the orch's driver-busy notice line grows by the quoted refusal and nothing else changes.
