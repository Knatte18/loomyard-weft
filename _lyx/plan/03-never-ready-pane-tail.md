# Card 3 — A never-ready teardown's error ends with the pane's last lines

**Create:**
- `plan:internal/shuttleengine#captureTail` -> `func captureTail(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine/startup_test.go`

**Uses:**
- `internal/shuttleengine#paneTailLines`
- `internal/shuttleengine#withPaneTail`

**Intent:**
Decision D4, with its added scope statement: with card 2, the producer's message then shows the screen that ended a never-ready start without opening the trace or the saved capture file.

`captureTail(capture)` returns capture's last `paneTailLines` non-blank lines, each right-trimmed, joined by newlines: the cut `paneTail` does today.
`paneTail` keeps its capture and its capture-failed sentence and calls `captureTail` for the cut, so a send failure's tail is unchanged.

`Run.abandonStartup` ends its returned error with `. The pane's last lines:\n<captureTail(run.lastStartupCapture)>`, the wording `withPaneTail` uses, exactly when all three hold:
- `cause` is nil;
- the provider never reached `StartupReady`, read as `run.state.Started` being false;
- `run.lastStartupCapture` is non-empty.
That is `awaitStartup`'s not-ready exits: the pane-died answer, the startup-window expiry, the run-deadline expiry and the tick-cap exit.
The tail is added whether or not the capture file was saved.

Nothing is added in every other case:
- A non-nil cause: busy, in-draft and unlanded send causes already end with the pane's last lines through `withPaneTail`, and the reed-status retry exits, the skill-loader lookup, the prompt-offset persist and a raw key or paste delivery error report a mechanism failure with no screen, so no cause gets a second tail.
- The teardown of a pane that dies during the skill-load turn: its cause is nil, but `run.state.Started` is true, and its last startup capture is the ready screen from before the death.
- A start whose every capture failed has no capture to cut.

The function keeps its single `return result, fmt.Errorf(format, args...)`, growing `format` and `args` before it, so `completionsignal_enforcement_test.go`'s `abandonStartup [Errorf]` count stays 1.
Its doc comment states the tail and the three conditions.

Tests, all in `startup_test.go`:
- `TestStartup_PaneNotLiveMidStartup`'s "pane goes not-live" row asserts the error ends with `"The pane's last lines:\n" + lastCapture`.
- `TestStartup_SkillLoadTurns`'s "pane that dies during a load turn is a died startup" row asserts the error does not contain `The pane's last lines:`.
- `TestStartup_CaptureAlwaysErroringUntilWindowExpires` asserts the error does not contain `The pane's last lines:`.
- `TestStartup_SendFailuresTearDown`'s "a skill-load send that stays busy" row asserts `The pane's last lines:` occurs exactly once in the error.

**ImpactSummary:** Only `abandonStartup`'s error text grows, on its never-ready, no-cause exits; `paneTail`'s output is unchanged.

**Commit:** 3: A never-ready teardown's error ends with the pane's last lines
