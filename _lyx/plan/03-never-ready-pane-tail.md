# Card 3 — A never-ready teardown's error ends with the pane's last lines

**Create:**
- `plan:internal/shuttleengine#lastPaneLines` -> `func lastPaneLines(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_UndismissableGateUntilWindowExpires`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_NeverReady`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine#TestStartup_CaptureAlwaysErroringUntilWindowExpires`
- `internal/shuttleengine#TestStartup_MechanismFailure`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#paneTailLines`

**Intent:** Decision D4, with its parent-recorded scope: the error of a start whose provider never reached `StartupReady`, torn down with no cause, ends with the last lines of its last startup capture.

`lastPaneLines` returns the last `paneTailLines` non-blank lines of a capture string, each right-trimmed, joined by newlines: the body `paneTail` has today after its `CapturePane` call.
`paneTail` keeps its capture-failure sentence and returns `lastPaneLines(capture)` otherwise, so a send failure's tail is unchanged.

`Run.abandonStartup` appends `. The pane's last lines:\n` and `lastPaneLines(run.lastStartupCapture)` at the very end of its returned error, after the removal note, when all three hold: `cause` is nil, `run.state.Started` is false, and `run.lastStartupCapture` is non-empty.
Those are `awaitStartup`'s never-ready exits: the pane dying before ready, the startup window or tick cap expiring, and the run deadline arriving first.
A non-nil cause gets no tail: the reed-status retry exits, the skill-loader lookup, the prompt-offset persist and a raw delivery error carry a mechanism failure with no screen, and a busy or unlanded send cause already ends with the pane's last lines through `withPaneTail`.
The skill-load death passes a nil cause but runs after `checkLivenessTick` set `Started`, and its last startup capture is the ready screen from before the death, so it gets no tail.
`abandonStartup`'s doc comment states the tail and its three conditions.
`internal/shuttleengine/doc.go`'s not-ready teardown sentence says the error of a provider that never came up ends with the last capture's lines.

`skillReed` gains a `Screen string` field, and `skillReed.CapturePane` returns `Screen` ahead of the typed texts it returns today, so a test can give the startup probe a non-empty capture.

Tests:
- `TestStartup_PaneNotLiveMidStartup`'s `lastCapture` becomes more than `paneTailLines` non-blank lines with blank lines among them, and every row asserts the error ends with `. The pane's last lines:\n` plus its last `paneTailLines` non-blank lines; the capture-file assertion keeps comparing the whole capture.
- `TestStartup_UndismissableGateUntilWindowExpires` asserts the window-expiry error ends with the gate capture's lines.
- `TestStartup_RunDeadlineShorterThanWindow_NeverReady` gives its fake reed a booting-screen `CaptureQueue` and asserts the run-deadline error ends with that screen's lines.
- `TestStartup_TickCap`'s `NoOutputFiles_ErrNotStartedWithTeardown` subtest gives its fake reed a booting-screen `CaptureQueue` and asserts the tick-cap error ends with that screen's lines.
- `TestStartup_CaptureAlwaysErroringUntilWindowExpires` asserts the error carries no `The pane's last lines`, since no capture exists.
- `TestStartup_MechanismFailure` gains a row "reed stops tracking the strand after a capture": status queue live `strand-1` then a strand table without it, capture queue a booting screen, so the first tick records a capture and the next two fail not-tracked; it wants `errStrandNotTracked`, and every row asserts the error carries no `The pane's last lines`.
- `TestStartup_SkillLoadTurns`'s "pane that dies during a load turn is a died startup" row sets `Screen` to a ready screen through its configure hook and asserts the error carries no `The pane's last lines`.

**ImpactSummary:** `paneTail`'s output is unchanged, and `abandonStartup`'s error gains a suffix only on its cause-free never-ready exits; every caller matches it with `errors.Is` or a substring, and the one exact-text assertion on it (`no pane capture was saved`) still holds.

**Commit:** 3: A never-ready start's error ends with the pane's last lines
