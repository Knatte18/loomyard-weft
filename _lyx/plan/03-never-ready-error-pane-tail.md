# Card 3 — never-ready-error-pane-tail

**Edit:**
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#withPaneTail`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_MechanismFailure`
- `internal/shuttleengine#TestStartup_CaptureAlwaysErroringUntilWindowExpires`

**Create:**
- `plan:internal/shuttleengine#captureTail` -> `func captureTail(capture string) string`
- `plan:internal/shuttleengine#paneTailHeading` -> `const paneTailHeading = "The pane's last lines:"`

**Uses:**
- `internal/shuttleengine#paneTailLines`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`

**Intent:** D4 as restated by the parent's addition: a never-ready teardown's error ends with the pane's last lines when a startup capture exists.

`captureTail` is the cut `paneTail` does today over a capture string: the last `paneTailLines` non-blank lines, right-trimmed, joined by newlines.
`paneTail` captures the pane and returns `captureTail` of it, keeping its failed-capture sentence.
`paneTailHeading` is the heading `withPaneTail` writes before the lines, and `withPaneTail` uses the constant in place of its literal so the two sites cannot drift.

`abandonStartup` appends `". " + paneTailHeading + "\n" + captureTail(run.lastStartupCapture)` to the returned error when `cause` is nil, `run.state.Started` is false and `run.lastStartupCapture` is non-empty;
`Started` is set in memory by `checkLivenessTick` on the ready probe before any later step runs, so false here means the provider never reached `StartupReady`, which is the trigger the record names.
`awaitStartup`'s never-ready exits (the window and tick-cap exits, the run-deadline exit, and the pane going not-live) all pass a nil cause with `Started` false and so get the tail;
the reed-status retry exits carry a cause and get none;
the teardown of a pane that dies during the skill-load turn has a nil cause but `Started` true, so its ready-screen capture is not appended;
a never-ready exit with no successful capture appends nothing and keeps saying no capture was saved.
The error still wraps `ErrNotStarted`, and the log lines are unchanged.
The method's doc comment states the rule in the same terms.

Tests, as assertions in the existing tests:
`TestStartup_PaneNotLiveMidStartup` asserts on every row that the error's text ends with `paneTailHeading`, a newline and the last capture's non-blank lines;
`TestStartup_SkillLoadTurns`'s row "pane that dies during a load turn is a died startup" asserts the error does not contain `paneTailHeading`;
`TestStartup_MechanismFailure` asserts on its status-cap rows that the error does not contain `paneTailHeading`;
`TestStartup_CaptureAlwaysErroringUntilWindowExpires` asserts the error does not contain `paneTailHeading` and still says no pane capture was saved.

**ImpactSummary:** One teardown method and one shared tail cut; every never-ready start error grows a tail, send failures keep their wording through the shared constant, and no caller matches on the message text.
