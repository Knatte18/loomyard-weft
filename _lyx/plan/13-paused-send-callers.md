# Card 13 — the seat notice sender retries a paused send, and the operator verbs refuse it with the pause's way forward

**Edit:**
- `internal/seatengine#noticeSender`
- `internal/seatengine#noticeSender.deliver`
- `internal/seatengine#newNoticeSender` -> `func newNoticeSender(chairOutputs []string, retry time.Duration, after func(time.Duration) <-chan time.Time) *noticeSender`
- `internal/seatengine#Engine`
- `internal/seatengine#New`
- `internal/seatengine#Engine.Run`
- `internal/seatengine#Engine.Resume`
- `internal/seatengine#engineFixture`
- `internal/seatengine#TestEngine_Run_AdvisorDeathSendsOneNoticeOnceTheChairTakesIt`
- `internal/loomcli#loomCLI.resumeParkedDriver`
- `internal/loomcli#TestResumeParkedDriver`
- `internal/shuttlecli#shuttleCLI.sendCmd`
- `internal/shuttlecli#TestInterruptAndSendCmd_FailWithTheGuardsMessage`

**Uses:**
- `plan:internal/shuttleengine#ErrPaused`
- `plan:internal/shuttleengine#PausedError`
- `internal/pauserecord#CoveredError`
- `internal/seatengine#fakeShuttle`
- `internal/loomcli#fakeDriverSender`
- `internal/loomcli#newResumeTestReceiver`

**Intent:**
D2 for the seat advisor notices, `lyx loom resume`'s driver resume line and `lyx shuttle send`.

`noticeSender.deliver` retries a send refused with `ErrPaused` instead of dropping the line: while the refusal is `ErrPaused` the wait between attempts doubles from the sender's retry interval up to five seconds (`PATTERN-daemon-wakeups`), and returns to the retry interval after any other held answer; the stop and all-outputs-exist ends are unchanged.
Its doc comment says a paused chair holds the line until the resume.
The wait between attempts goes through a new `after` field of `noticeSender`, the timer seam `newNoticeSender` takes, so a test can read each wait; `Engine` gains the field `noticeAfter`, which `New` sets to `time.After` and `Engine.Run` and `Engine.Resume`, the two callers of `newNoticeSender`, pass on.
`engineFixture` sets `noticeAfter` to a recorder that answers at once and keeps each duration it was asked for.
`TestEngine_Run_AdvisorDeathSendsOneNoticeOnceTheChairTakesIt` gains a column of the waits it expects and two rows: with the row's retry interval set to two seconds, a chair that answers four `*shuttleengine.PausedError` refusals before it takes the lines waits 2 s, 4 s, 5 s and 5 s, and delivers both notices once; a chair that answers a `PausedError`, then `ErrSessionBusy`, then a `PausedError` waits the retry interval each time, the busy answer resetting the doubling.

`resumeParkedDriver`, on a send error wrapping `ErrPaused`, makes no further attempt, keeps the park marker and the resume-report file, and returns `pauserecord.CoveredError(pe.Entry, pe.Slug, fmt.Sprintf("run %q again", retry))` behind `loom: the driver's pane is held by a pause: `, so the refusal's way forward names `lyx resume` and the retry; it is an operator verb, so naming `lyx resume` is right there.
`TestResumeParkedDriver` gains a row whose sender answers a `*shuttleengine.PausedError`: one send, no wait, the error names `"lyx resume"` and the retry verb and not `may have resumed`, and both files stay.

`sendCmd` keeps its coverage check by its own slug before the send, and renders a send refused with `ErrPaused`, a target another slug's entry covers, as `pauserecord.CoveredError(pe.Entry, pe.Slug, "re-run lyx shuttle send")`, the same way forward its own check gives.
`TestInterruptAndSendCmd_FailWithTheGuardsMessage` gains a row and the table the columns it needs: a geometry to tell, through both `cli.pause` and `runner.SetPause`, since the refusal comes from the runner, and a `Spec.NameOverride` naming the started strand in a slug.
The row tells a geometry with an empty slug over a record holding a slug entry for `p1`, starts the strand named `ly:p1:impl`, and the send fails naming `lyx resume` and `lyx shuttle send` with nothing typed, the CLI's own check by its empty slug reading uncovered.

**ImpactSummary:** three callers of shuttle's send each gain one `ErrPaused` branch; the seat sender keeps its one-writer loop, and the two operator verbs keep every other refusal as it is.
