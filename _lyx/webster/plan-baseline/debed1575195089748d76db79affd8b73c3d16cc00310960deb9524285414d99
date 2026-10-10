# Card 8 — readiness-reading

**Create:**
- `plan:internal/shuttleengine#readiness` -> `type readiness struct`
- `plan:internal/shuttleengine#boxReading` -> `type boxReading struct`
- `plan:internal/shuttleengine#readBox` -> `func readBox(engine Engine, capture string) boxReading`
- `plan:internal/shuttleengine#judgeReadiness` -> `func judgeReadiness(state SessionState, unmatched bool, box boxReading) readiness`
- `plan:internal/shuttleengine#sessionReadiness` -> `func sessionReadiness(sc sendContext, cycler SessionCycler, hold *turnStartHold, capture string) readiness`
- `plan:internal/shuttleengine#Runner.holdFor` -> `func (r *Runner) holdFor(guid string) *turnStartHold`

**Edit:**
- `internal/shuttleengine#IdleProbe`
- `internal/shuttleengine#Runner`
- `internal/shuttleengine#Runner.SessionIdle`
- `internal/shuttleengine#sendContext`
- `internal/shuttleengine#Run.newSendContext`
- `internal/shuttleengine#Runner.newSendContext`
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#busyReading`
- `internal/shuttleengine#ErrSessionBusy`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/doc.go`

**Intent:** One readiness reading, built on the hook-derived session state, gates every orch injection and every verified send (D7).
`boxReading` is what the pane says about the input box: whether the engine reads one (`InputBoxReader` present and a box found), its text, and whether the pane is too short; `readBox` fills it from a capture, with no box and not too short for an engine without the reader.
`judgeReadiness` is pure and total over the fold's states, reading the state's name and cause and never the newest signal: ready on `idle-done`, `idle-stalled` of any cause, `asking` with cause `awaiting-input` and `busy` with cause `background`, provided no turn start is unmatched and the box, where one is read, is blank; held with a reason string on `busy`/`turn`, an unmatched turn start, `asking`/`ask`, `dead`, a non-blank box (reason naming a draft) and a pane too short; `unknown` of any cause yields `readiness.unknown` with the cause.
`readiness` carries `ready`, `reason`, `unknown` and `cause`.
`sessionReadiness` composes the reading for one strand: it folds the session state from `sc.state` through `readRunSessionState` at `sc.clock.Now()`, asks `unmatchedTurnStart` and the hold whether a turn start still holds (the hold released as today by a later turn end, a reported interrupt, or the pane reading idle through `cycler.IdleSession` on every poll for `turnStartIdleOverride`, logged at Warn), and reads the box from the capture.
`sendContext` gains `state RunState`, which both constructors fill, so the reading has the run's events path, interactivity and output files.
`busyReading` takes the reading first: a ready reading returns `""`, a held one its reason, and an `unknown` one falls back to today's pane logic (the ready needle, `IdleSession`, the unmatched-turn-start hold), logging the cause at Debug; `awaitIdleSession` is unchanged in shape and keeps one hold across its polls.
`ErrSessionBusy`'s comment names the reading.
`Runner` gains a mutex-guarded map of `turnStartHold` by strand guid, reached through `holdFor`, so the orch watcher's probes keep one hold across its ticks as the send keeps one across its polls.
`Runner.SessionIdle` computes the reading through `sessionReadiness` over the strand's run state and one capture: a ready or held reading becomes `IdleProbe{Idle, Reason}`, with `TooShort` set when the held reason is the short pane; an `unknown` reading falls back to today's probe (`IdleSession`, `PaneTooShort`) and logs the cause at Debug.
When the reading is ready and the pane shows the running-turn needle or no box, the probe still answers idle and logs the disagreement with the pane's last lines at Debug, through `paneTail`.
Bound: this downgrades the pane probe, so a ready reading beside a needle or a missing box lets an injection reach a screen state the hooks do not report, such as a menu; it is bounded by the verified send's landed check, which retries or reports a submission that did not land, and by the fact that only the watcher's own commands and prompts are typed through this gate, never operator text.
`IdleProbe` gains `Reason string`, the held reason, empty when idle.
`judgeReadiness` has no test of its own: it is covered through the two public surfaces it serves.
`session_test.go`'s `TestRunner_SessionIdle_ReturnsScriptedClassification` becomes the table over `Runner.SessionIdle`, driven over a fake reed and an engine with the signal parser, the cycler and the box reader (extending `fakes_test.go`), one row per state and cause in the acceptance list (`idle-done`, `idle-stalled`, `asking`/`awaiting-input`, `busy`/`background` ready; `busy`/`turn`, `asking`/`ask`, `dead` held with their reasons; `unknown` falling back to the pane probe), plus rows for a draft held with its reason, a pane too short, a hook turn end beside a pane needle that reads idle and logs the disagreement, a turn start with no turn end held and then released by a later turn end, an interrupt report and the idle override across polls, a session-start line after a turn end that leaves it idle, and its existing rows for an engine without the parser, which fall back to the pane probe.
`run_test.go`'s `TestSend_WaitsForIdleSession` keeps to what is specific to the send: its existing draft and unmatched-turn-start rows are repinned to the reading's held reasons, and it gains one row, a hook turn end beside a needle that types at once.
`doc.go`'s verified-send section states the reading and its fallback, and its session-state section replaces the statement that the state runs in shadow mode with no consumer acting on it with the reading's two consumers, `Runner.SessionIdle` and the verified send's idle wait.

**ImpactSummary:** The idle probe and the send's idle wait change their decision source behind unchanged signatures; two struct types gain a field, and the only new seam state is the per-strand hold on the runner.
