# Card 3 — never-ready-pane-tail

**Create:**
- `plan:internal/shuttleengine#lastLines` -> `func lastLines(capture string) string`
- `plan:internal/shuttleengine#Run.abandonNeverReady` -> `func (run *Run) abandonNeverReady(outcome Outcome) (Result, error)`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#ErrNotStarted`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_NeverReady`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine/doc.go`

**Intent:** A never-ready start's error ends with the last non-blank lines of its last startup capture (decision D4 and its tail-scope addendum).
`lastLines` is the cut `paneTail` does today after its capture: trim each non-blank line's trailing whitespace, keep the last `paneTailLines`, join with newlines.
`paneTail` calls it, so a send failure's tail is unchanged.
`abandonNeverReady` calls `abandonStartup(outcome, nil)` and, when `run.lastStartupCapture` holds a non-blank line, returns the error extended with `". The pane's last lines:\n"` and `lastLines(run.lastStartupCapture)`, wording `withPaneTail` uses, so the message ends with the tail and still wraps `ErrNotStarted`.
With no capture, or a capture of blank lines only, the error is `abandonStartup`'s own, unchanged.
`awaitStartup` calls `abandonNeverReady` at its three nil-cause exits, where the provider never reached `StartupReady`: the pane-died answer, the run-deadline timeout, and the tick-cap exit.
Its reed-status retry exits keep calling `abandonStartup` with their cause and get no tail, and so do the exits in `start` (skill-loader lookup, skill-load send, prompt-offset persist, prompt delivery) whose causes already end with a tail or report a mechanism failure with no screen.
The teardown of a pane that dies during the skill-load turn is `start`'s own nil-cause call (`run.go:585`): the provider had reached ready, its last capture is the ready screen, and it must get no tail, which is why the tail lives in `abandonNeverReady` and not in `abandonStartup`.
Nothing is skipped or let through: the change only appends text to an error already returned.
The `awaitStartup` return-mapping comment gains a sentence naming the tail.
`abandonStartup`'s comment, which says `cause` is nil when the provider never reached `StartupReady`, is reworded to say a nil cause also covers the skill-load death, and `ErrNotStarted`'s comment names `abandonNeverReady` beside `abandonStartup` for the error text.
`internal/shuttleengine/doc.go`'s startup paragraph gains a sentence that a never-ready start's error ends with the pane's last lines.
Rows, as additions to existing tests: `TestStartup_PaneNotLiveMidStartup` asserts that its "pane goes not-live" row's error ends with the capture it queued (`still booting...`) after `The pane's last lines:`; `TestStartup_TickCap` ("NoOutputFiles") and `TestStartup_RunDeadlineShorterThanWindow_NeverReady` ("StartGated") queue a non-blank pane capture on their fake reed and assert the error ends with it, reaching the tick-cap and run-deadline exits; `TestStartup_SkillLoadTurns` gains a `capture` table field the row "pane that dies during a load turn is a died startup" sets, queued on the reed in its configure callback, and asserts the error does not contain `The pane's last lines:` (the capture is non-blank, so absence comes from the exit and not from an empty capture); `TestStartup_SendFailuresTearDown` asserts that the error of each row ending in a busy or unlanded send holds `The pane's last lines:` at most once, so no cause gets a second tail.
The reed-status retry exits get no new assertion: `checkLivenessTick` returns their errors before its `CapturePane` call, so `lastStartupCapture` is empty there and an absence check could not fail.

**ImpactSummary:** `awaitStartup` is called only from `start`; `paneTail` keeps its signature and callers; the only visible change is a longer error on three never-ready exits, with the existing tests that check `errors.Is` untouched.
