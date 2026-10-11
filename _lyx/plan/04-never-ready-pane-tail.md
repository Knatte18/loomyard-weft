# Card 4 — A never-ready start's error ends with the pane's last lines

**Create:**
- `plan:internal/shuttleengine#lastLines` -> `func lastLines(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`

**Uses:**
- `internal/shuttleengine#paneTailLines`

**Intent:**
Decision D4, with the parent's later scope statement.
With card 3, the producer's message then shows the screen that ended a never-ready start without opening the trace or the saved capture file.

`lastLines` holds the cutting `paneTail` does today: the last `paneTailLines` non-blank lines of capture, each right-trimmed of spaces, tabs and carriage returns, joined by newlines.
`paneTail` captures the pane and returns `lastLines` of the capture, or its capture-failed sentence as today; its output is unchanged.

`Run.abandonStartup` appends `. The pane's last lines:\n<lastLines(run.lastStartupCapture)>` to its returned error, the same suffix shape `withPaneTail` writes, exactly when all three hold:
- `cause` is nil;
- the provider never reached `StartupReady`: `run.state.Started` is false;
- a startup capture exists: `run.lastStartupCapture` is non-empty.
That is `awaitStartup`'s not-ready exits: the pane dying before ready, the startup window or run deadline closing, and the tick cap.
The reed-status retry exits are never-ready but carry a cause, so they get no tail.
A pane that dies during the skill-load turn has a nil cause but `Started` true, since its last startup capture is the ready screen from before the death, so it gets no tail.
When a cause is present nothing is added: a busy, in-draft or unlanded send cause already ends with the pane's last lines through `withPaneTail` or the busy wait, and the remaining causes (the reed-status retry exits, the skill-loader lookup, the prompt-offset persist and a raw key or paste delivery error) report a mechanism failure with no screen.
Update `abandonStartup`'s doc comment to state when the tail is appended.

Tests:
- `TestStartup_PaneNotLiveMidStartup`: the `pane goes not-live` row asserts the error ends with `The pane's last lines:\n` followed by the row's last capture; add a row whose last capture holds more non-blank lines than `paneTailLines`, with blank lines between, asserting the tail is the last `paneTailLines` of them in order.
- `TestStartup_SkillLoadTurns`: the `pane that dies during a load turn is a died startup` row asserts the error does not contain `The pane's last lines:`.

No existing test asserts `abandonStartup`'s full error text; tests match it by `errors.Is` or by substrings that keep holding.

**ImpactSummary:** `paneTail`'s output is unchanged; `abandonStartup`'s error gains a pane tail on the never-ready nil-cause exits only.

**Commit:** 4: End a never-ready start's error with the pane's last lines
