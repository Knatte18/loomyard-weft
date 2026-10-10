# Card 6 — seatengine-run

**Create:**
- `plan:internal/seatengine#Engine` -> `type Engine struct`
- `plan:internal/seatengine#New` -> `func New(shuttle Shuttle, geom Geometry) *Engine`
- `plan:internal/seatengine#SeatResult` -> `type SeatResult struct`
- `plan:internal/seatengine#Result` -> `type Result struct`
- `plan:internal/seatengine#ErrSeatNotStopped` -> `var ErrSeatNotStopped = errors.New("seatengine: a seat could not be stopped")`
- `plan:internal/seatengine#seatRun` -> `type seatRun struct`
- `plan:internal/seatengine#Engine.Run` -> `func (e *Engine) Run(table Table) (Result, error)`
- `plan:internal/seatengine#Engine.startSeat` -> `func (e *Engine) startSeat(table Table, seat Seat, names map[string]string, failed []string) (Handle, error)`
- `plan:internal/seatengine#Engine.join` -> `func (e *Engine) join(table Table, chair Handle, advisors []*seatRun, sender *noticeSender) (Result, error)`
- `plan:internal/seatengine#checkName` -> `func checkName(handle Handle, want string) error`
- `plan:internal/seatengine#stopSeat` -> `func stopSeat(handle Handle) error`
- `plan:internal/seatengine#noticeSender` -> `type noticeSender struct`
- `plan:internal/seatengine#newNoticeSender` -> `func newNoticeSender(chairOutputs []string, retry time.Duration) *noticeSender`
- `plan:internal/seatengine#noticeSender.queue` -> `func (s *noticeSender) queue(line string)`
- `plan:internal/seatengine#noticeSender.start` -> `func (s *noticeSender) start(chair Handle)`
- `plan:internal/seatengine#noticeSender.stop` -> `func (s *noticeSender) stop()`
- `plan:internal/seatengine#advisorNotice` -> `func advisorNotice(name string, outputs []string, outcome shuttleengine.Outcome) string`
- `internal/seatengine/engine_test.go`
- `internal/seatengine/seatfake_test.go`

**Edit:**
- `internal/seatengine/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/seatengine#seatSpec`
- `plan:internal/seatengine#composePrompt`
- `plan:internal/seatengine#seatValues`
- `plan:internal/seatengine#strandNames`
- `internal/burlerengine#Engine.Run`
- `internal/burlerengine#Engine.join`
- `internal/shuttleengine#ErrNotStarted`
- `internal/shuttleengine#ErrSessionBusy`
- `internal/shuttleengine#ErrSubmissionNotLanded`

**Intent:** Run a table fresh (D5, D7, D8 start order, D9) in `engine.go` and `notice.go`.
`Engine` holds the shuttle, the geometry and an unexported `noticeRetry` interval that `New` sets to a production default and in-package tests shorten (PATTERN-test-speed).
`Result` carries the chair's `shuttleengine.Result` and `ChairOutputs`, plus one `SeatResult` per advisor: `Name`, `StrandGUID`, `RunDir`, `StartError`, `Outcome` and `Notified`; a Go-side notice is recorded on the advisor's result so the adapter and tests can read it.
`Engine.Run` validates the table, forms the names, then starts the advisors in table order and the chair last through `startSeat`, which composes the prompt and spec from card 5, calls `StartGated` (the chair with the table's gate, an advisor ungated) and runs `checkName`: a strand whose formed name differs from the expected one, reed's numbering of a held name, is a start error naming the holder and the way forward `lyx reed remove --name <name>`.
An advisor that fails to start is recorded with its `StartError`, its name joins the chair's `failed_advisors`, and no notice is sent for it; each started advisor's `Wait` is joined in its own goroutine from the moment it starts, and its end is queued on the notice sender as an `advisorNotice` line, one line naming the advisor, saying it ended and that its outputs stand where they exist.
A chair that fails to start stops every started advisor and returns the error, wrapping `shuttleengine.ErrNotStarted` when the provider never came up; the queue is dropped with it.
The notice sender is one goroutine per chair, started with the chair's handle: it drains the queue one line at a time through the handle's `Send`, checks that not every chair output exists before each send, retries a `shuttleengine.ErrSessionBusy` or `ErrSubmissionNotLanded` line after `noticeRetry` until it lands, the chair's outputs all exist or the sender is stopped, and logs a line that never lands rather than erroring; two queued lines are typed one after the other, never interleaved.
`join` waits on the chair, stops the sender, then stops every live advisor through `stopSeat`, which calls the handle's `Stop` and on failure returns an error wrapping `ErrSeatNotStopped` naming `lyx reed remove <guid>` and a re-step as the way forward, never retrying; the result carries the chair's outcome, gate and `NotStarted` as shuttle reported them, and the engine judges nothing about finishing itself (PATTERN-completion-signal).
`seatfake_test.go` holds the handle-shaped fake shuttle: scripted start errors, per-seat results released by the test, recorded sends with a scriptable busy answer, recorded stops with a scriptable failure, and a formed-name override.
`engine_test.go` covers the running acceptance list: start order; chair Done, failing gate, died, timeout and never-started results with advisors stopped; chair start failure stopping the started advisors; advisor start failure recorded, named in the chair's prompt and sending nothing; an advisor death sending exactly one line once the chair is idle, with a busy chair delaying it without blocking the join, and the chair settling or its outputs appearing ending the retries; two advisors ending together sending two whole lines in sequence; an advisor ending while the chair is still starting notified once the chair's handle exists; a Done advisor kept until the chair's terminal result; and a failed stop returning the way forward unretried.
Extend `doc.go` with the start order, failure and hang rules, the notice sender and the stop rule.
Raise the `internal/seatengine` budget row by the tests this card adds.

**ImpactSummary:** doc.go grows; nothing outside the package is touched and no earlier seatengine symbol changes shape.
