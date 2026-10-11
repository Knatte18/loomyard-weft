# Card 2 — A never-ready start's error ends with the pane's last lines

**Create:**
- `plan:internal/shuttleengine#lastLines` -> `func lastLines(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#flakyStatusReed`
- `internal/shuttleengine#flakyStatusReed.Status`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_MechanismFailure`
- `internal/shuttleengine#TestStartup_CaptureAlwaysErroringUntilWindowExpires`

**Uses:**
- `internal/shuttleengine#paneTailLines`

**Intent:**
D4, with the parent's addition that states its scope as never-ready with no cause.

`lastLines` in `send.go` is the pure half of `paneTail`: it returns the last `paneTailLines` non-blank lines of capture, each right-trimmed of spaces, tabs and carriage returns, joined by newlines.
`paneTail` keeps its signature and behavior and calls it on the capture it took.

`abandonStartup` appends the tail only when all three hold: cause is nil, `run.state.Started` is false (the provider never reached `StartupReady`, which `checkLivenessTick` records in memory before its best-effort save), and `run.lastStartupCapture` is non-empty.
The returned error then ends with `. The pane's last lines:\n` followed by `lastLines(run.lastStartupCapture)`, the same wording `withPaneTail` uses for a send failure.
The tail extends the function's one `format`/`args` pair the way the cause already does, so `abandonStartup` keeps its single `fmt.Errorf` call, the count `completionsignal_enforcement_test.go` pins as `"abandonStartup [Errorf]": 1`.
That covers `awaitStartup`'s nil-cause exits: a pane that went not-live before ready, an expired startup window, the run deadline and the tick cap.
The reed-status retry exits carry a cause and get no tail.
The skill-load death in `loadSkillsThenPrompt` passes a nil cause on a started run, whose last capture is the ready screen from before the death, so it gets no tail.
Every other cause gets nothing added: busy and unlanded send causes already end with the pane's last lines through `withPaneTail`, and the rest report a mechanism failure with no screen.
`abandonStartup`'s doc comment states the tail and its condition.

Tests.
`TestStartup_PaneNotLiveMidStartup`'s `lastCapture` becomes a capture of `paneTailLines` + 5 numbered lines (`line 1` …) with one blank line among the early ones, built once in the test.
Its "pane goes not-live" row (the one with `wantCapture`) keeps asserting `startup-capture.txt` holds the whole capture, and also asserts the error ends with `"The pane's last lines:\n"` followed by the capture's last `paneTailLines` non-blank lines joined by newlines.
It fails today because no tail is added, and it fails if `lastLines` drops the cut to `paneTailLines`.
`TestStartup_CaptureAlwaysErroringUntilWindowExpires` also asserts the error does not contain `The pane's last lines`: a never-ready start with no capture gets no tail.
`skillReed` gains a `StartupCapture string` field that its `CapturePane` writes ahead of the typed texts, empty by default so every other row's capture is unchanged.
`TestStartup_SkillLoadTurns`' configure callback sets it to a ready screen for the `wantDied` row ("pane that dies during a load turn is a died startup"), so `run.lastStartupCapture` is non-empty when the pane dies, and the row asserts the error does not contain `The pane's last lines`.
It fails if the tail's condition drops the `Started` check.
`flakyStatusReed` gains a `passFirst int` field: its first `passFirst` `Status` calls delegate to the embedded `fakeReed`, the next `failFirst` return the scripted error, and later calls delegate again.
`TestStartup_MechanismFailure` gains the row "status errors after a pending capture exhaust the retry cap with no tail": the inner `fakeReed`'s `StatusQueue` is `[]reedengine.StatusResult{{Strands: []reedengine.StrandStatus{{GUID: "strand-1", PaneID: "%1", Live: true}}}}` (its zero value reads untracked and would skip the capture) and its `CaptureQueue` holds `still booting...`, `passFirst` is 1 and `failFirst` is `maxStatusRetries`, and the row asserts, beside the existing assertions, that the error does not contain `The pane's last lines`.
Its first probe records the capture before the status errors start, so the row fails if the tail's condition drops the nil-cause check.
The two existing rows set no `passFirst`, so they behave as today.

**ImpactSummary:** `paneTail`'s callers see no change; `abandonStartup`'s error text grows a tail on the never-ready nil-cause exits only, which every caller reads through `errors.Is(err, ErrNotStarted)` and which reaches the producer through card 3.

**Commit:** 2: A never-ready start's error ends with the pane's last lines
