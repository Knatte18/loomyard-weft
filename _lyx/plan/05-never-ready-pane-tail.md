# Card 5 — A never-ready start's error ends with the pane's last lines

**Create:**
- `plan:internal/shuttleengine#captureTail` -> `func captureTail(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#Run.loadSkillsThenPrompt`

**Intent:**
Decision D4, with its scope as the parent restated it after the Discussion: the error of a start whose provider never reached `StartupReady`, torn down with no cause, ends with the last lines of its last startup capture, so the producer message (Cards 3 and 4) shows the screen that ended the start.

`captureTail`, in `send.go`, returns the last `paneTailLines` non-blank lines of a capture, each right-trimmed, joined by newlines: the cut `paneTail` makes today.
`paneTail` keeps its signature and its capture-failure sentence and cuts through `captureTail`.

`Run.abandonStartup` adds the tail when `cause` is nil, `run.state.Started` is false and `captureTail(run.lastStartupCapture)` is non-empty.
It appends `. The pane's last lines:\n<tail>` after the remove note, the wording `withPaneTail` uses; the error still wraps `ErrNotStarted`.
That set is exactly `awaitStartup`'s three not-ready exits with a nil cause: the pane gone not-live before ready, the run deadline, and the tick cap; a startup window that expires reaches the pane-not-live exit's call through `classifyStartupWindow`.
`run.state.Started` is what tells these apart from the pane that dies during a skill-load turn in `loadSkillsThenPrompt`: that call also passes a nil cause, but `checkLivenessTick` set `Started` on the ready probe, and its last startup capture is the ready screen from before the death, so it gets no tail.
A call with a cause adds nothing: busy, in-draft and unlanded send causes already end with the pane's last lines through `withPaneTail`, and the reed-status retry exits, the skill-loader lookup, the prompt-offset persist and a raw key or paste delivery error report a mechanism failure with no screen.
No capture, or one with no non-blank line, adds nothing, as the existing "no pane capture was saved" note already says.
`abandonStartup`'s doc comment states when the tail is added.

`doc.go`'s startup paragraph says a never-ready start's error ends with the last lines of its last startup capture.

Tests:
- `TestStartup_PaneNotLiveMidStartup`: the "pane goes not-live" row asserts the error ends with `"The pane's last lines:\n" + lastCapture`; a new row whose last capture holds more than `paneTailLines` non-blank lines with blank lines between them asserts the error ends with exactly the last `paneTailLines` of them.
- `skillReed` gains a `Screen string` field, and its `CapturePane` returns `Screen` followed by the texts typed so far, so the typed texts the skill sends look for stay in the capture; an empty `Screen` keeps today's capture, so every other skill row is unchanged.
  Without it the ready probe's capture is empty, and a no-tail assertion could not fail.
- `TestStartup_SkillLoadTurns` gains a `screen` row field, applied to `skillReed.Screen` in its configure closure; the "pane that dies during a load turn is a died startup" row sets a non-blank ready screen and asserts the error does not contain `The pane's last lines`.

**ImpactSummary:** One package-private teardown's error text grows a tail on its three never-ready nil-cause exits and `paneTail` moves its cut into a helper with identical output; callers match `ErrNotStarted` with `errors.Is`, and `RunGated`'s `StartError` (Card 3) is the one carrier that now carries the screen.
