# Card 14 — batten retries a busy driver pane and notices the orch once per stall

**Create:**
- `plan:internal/battenshed#ErrChildDriverBusy` -> `var ErrChildDriverBusy = errors.New("battenshed: the task worktree's driver pane is busy")`
- `plan:internal/battenshed#innerRunProducer.noticeDriverBusy` -> `func (p *innerRunProducer) noticeDriverBusy(ctx context.Context, w *childWait)`

**Edit:**
- `internal/battenshed#innerRunProducer.awaitingStep`
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#childWait`
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#TestInnerRun_AwaitingNotParkedRetriesWithoutMarkerThenResumes`
- `internal/battenshed/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/battenshed#innerRunProducer.deliver`
- `internal/battenshed#renderNotice`
- `internal/battenshed#noticeLine`
- `internal/battenshed#ErrChildNotParked`

**Intent:**
Make a draft in a parked driver's pane delay batten's resume instead of failing batten (decision D9).

`ErrChildDriverBusy` is the sentinel an `InnerRunDeps.Spawn` closure wraps when the child's bootstrap refused with `shedrun.StartDriverBusyKind`; `InnerRunDeps.Spawn`'s doc comment names it beside `ErrChildNotParked`.

In `awaitingStep`, a spawn error wrapping `ErrChildDriverBusy` is handled exactly like `ErrChildNotParked`: the decision stays unacted (no decision-acted marker), `w.resumeRetryAt` moves one `noticeProbe` ahead, the first such refusal in the wait is logged at Info, and the step reports, then returns nil so the wait goes on.
The report reads `the <kind> at <time> is not acted on yet: the child's driver pane holds a draft or the operator is typing in it; clear the draft in the driver's pane and batten resumes it`, followed by the child's own refusal reason, which quotes the draft.
`childWait` gains `busyLogged bool` beside `notParkedLogged`; the retry-gate report before the spawn names the busy reason while the last refusal was busy.

`noticeDriverBusy` queues one orch notice per stall: `innerRunProducer` gains `driverBusyNoticed bool`; on a busy refusal with it false, the step renders a notice through `renderNotice` with the condition `driver pane busy: clear the draft in the driver's pane; batten resumes the child` and delivers it through `deliver` under a key built from that condition and the child's state, setting the flag once `deliver` reports it queued; a resume that succeeds clears the flag.
The notice never carries the spawn error or anything else from the child's pane, so no text from another pane is typed into the orch session.

Bound: like the not-parked retry it has no deadline; it ends when the resume lands, the child leaves awaiting, batten is paused or the batten run is stopped, and the notice plus `lyx batten status` keep the stall visible (R5); the retry keeps `noticeProbe` pacing (`PATTERN-daemon-wakeups`).

Test: `TestInnerRun_AwaitingNotParkedRetriesWithoutMarkerThenResumes` becomes a table with a not-parked row and a busy row: a `Spawn` returning `ErrChildDriverBusy` twice then nil spawns once per `noticeProbe`, writes no decision-acted marker until the spawn succeeds, reports the draft and the remedy, and queues exactly one notice, which does not contain the refusal's draft text; a second stall after the success queues a second notice.
Raise `internal/battenshed`'s row in `cmd/lyx/testdata/test-budget.yaml` if a new top-level test is needed, and document the busy retry in `internal/battenshed/doc.go`.

**ImpactSummary:** The awaiting arm's spawn-error branch gains one case and a notice; batten's other arms and the not-parked retry are unchanged.
