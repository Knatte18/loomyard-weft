# Card 3 — never-ready-error-ends-with-pane-tail

**Create:**
- `plan:internal/shuttleengine#captureTail` -> `func captureTail(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_CaptureAlwaysErroringUntilWindowExpires`
- `internal/shuttleengine#TestStartup_MechanismFailure`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`

**Uses:**
- `internal/shuttleengine#paneTailLines`
- `internal/shuttleengine#withPaneTail`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`

**Intent:** Decision D4 of the decision record: a never-ready teardown's error ends with the pane's last lines, taken from the last startup capture.

`captureTail` is the cutting half of today's `paneTail`: it drops blank lines, trims trailing whitespace from the rest and keeps the last `paneTailLines` lines, joined by newlines.
`paneTail` becomes `CapturePane` followed by `captureTail`, with its capture-failed sentence unchanged, so a send failure's tail and a never-ready tail are cut by one function.

`abandonStartup` appends the tail when all three hold: `cause` is nil, `run.state.Started` is false, and `run.lastStartupCapture` is non-empty.
The appended text is `. The pane's last lines:\n<captureTail(run.lastStartupCapture)>`, the same wording `withPaneTail` uses, placed after the capture and removal notes and still wrapping `ErrNotStarted`.
`Started` is what tells never-ready from the skill-load death: `checkLivenessTick` persists `Started` on the ready probe, so a pane that dies during the skill-load turn reaches `abandonStartup` with a nil cause and `Started` true, and its last startup capture, the ready screen from before the death, is not appended.
The reed-status retry exits carry a cause and get no tail; a busy, in-draft or unlanded send cause already ends with the pane's last lines through `withPaneTail`; every other cause (the skill-loader lookup, the prompt-offset persist, a raw key or paste delivery error) reports a mechanism failure with no screen.
So no error carries two tails, and the doc comment of `abandonStartup` states the rule and the `Started` reading.

Tests, each as an assertion in the existing test of that exit:
`TestStartup_PaneNotLiveMidStartup`'s "pane goes not-live" row asserts the error text ends with `The pane's last lines:\nstill booting...`, the capture the row already seeds;
`TestStartup_CaptureAlwaysErroringUntilWindowExpires` asserts the error text does not contain `The pane's last lines`, since no capture exists;
`TestStartup_MechanismFailure` asserts the same absence on both rows, since a cause is present;
`TestStartup_SkillLoadTurns`'s "pane that dies during a load turn is a died startup" row asserts the same absence, the nil-cause exit that is not never-ready.

**ImpactSummary:** paneTail keeps its signature and callers; abandonStartup's error text grows a suffix on one of its exits, read by the startup tests and, through card 2, by the single-LLM producer's message.
