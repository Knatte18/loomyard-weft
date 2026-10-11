# Card 3 — never-ready-pane-tail

**Create:**
- `plan:internal/shuttleengine#captureTail` -> `func captureTail(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_MechanismFailure`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`

**Intent:**
A never-ready start's error ends with the pane's last lines, so with card 2 the producer's message shows the screen that ended the start without opening the trace or the saved capture file (decision D4 and its later scope statement).

`captureTail` holds the cut `paneTail` does today: the last `paneTailLines` non-blank lines of a capture, each right-trimmed, joined by newlines.
`paneTail` keeps its capture and its capture-failure sentence and returns `captureTail` of the capture.

`Run.abandonStartup` adds the tail only when `cause` is nil, `run.state.Started` is false and `run.lastStartupCapture` is non-empty: the error then ends `. The pane's last lines:\n<captureTail(run.lastStartupCapture)>`, the wording `withPaneTail` uses.
`run.state.Started` is set in memory by `checkLivenessTick` on the ready probe before its save, so it tells a never-ready teardown from the skill-load death, whose nil cause comes after ready and whose last startup capture is the ready screen from before the death; that teardown gets no tail.
A teardown with a cause gets no tail: the busy, in-draft and unlanded send causes already end with the pane's last lines through `withPaneTail` or the busy wording, and the remaining causes (the reed-status retry exits, the skill-loader lookup, the prompt-offset persist and a raw key or paste delivery error) report a mechanism failure with no screen.
A never-ready teardown with no successful capture adds nothing.
The function keeps its one `fmt.Errorf` return, extending its format and arguments, since `completionsignal_enforcement_test.go` pins `abandonStartup`'s `Errorf` count at one.

`TestStartup_PaneNotLiveMidStartup` asserts the error ends with `The pane's last lines:\nstill booting...` on every row.
`TestStartup_MechanismFailure` gains a row whose first status lists `strand-1` live over a pending capture and whose later statuses list only another strand, so the start ends on the not-tracked retry exit with a capture recorded and errors wrapping `errStrandNotTracked`;
every row asserts the error carries no `The pane's last lines:` text.
`TestStartup_SkillLoadTurns`'s row for a pane that dies during a load turn asserts its error carries no `The pane's last lines:` text.

**ImpactSummary:** `paneTail`'s output is unchanged; only a never-ready, cause-free startup teardown's error text grows a trailing pane tail.

**Commit:** `3: A never-ready start's error ends with the pane's last lines`
