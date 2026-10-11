# Card 3 — A never-ready teardown's error ends with the pane's last lines

**Create:**
- `plan:internal/shuttleengine#lastPaneLines` -> `func lastPaneLines(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_CaptureAlwaysErroringUntilWindowExpires`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_MechanismFailure`

**Uses:**
- `internal/shuttleengine#paneTailLines`
- `internal/shuttleengine#withPaneTail`

**Intent:**
Decision D4, with the parent's later restatement of its scope: the tail goes only on a teardown whose provider never reached ready and that has no cause.
With card 2, the producer's message then shows the screen that ended the start, so the operator need not open the trace or the saved capture file.

- `lastPaneLines` returns the last `paneTailLines` non-blank lines of capture, each with trailing spaces, tabs and carriage returns trimmed, joined by newlines.
  This is the cut `paneTail` makes today.
  `paneTail` keeps its signature and its capture-failure sentence, and now calls `lastPaneLines` on the capture it takes.
- `abandonStartup` appends the tail only when all three hold: `cause` is nil, `run.state.Started` is false, and `lastPaneLines(run.lastStartupCapture)` is not empty.
  The tail is `. The pane's last lines:\n<lines>`, the wording `withPaneTail` uses, after the existing capture and removal notes.
  - These exits are never ready with no cause, and get the tail: `awaitStartup`'s pane-died exit, its run-deadline exit, its tick-cap exit, and a startup window that closes through `checkLivenessTick`.
  - The skill-load death in `loadSkillsThenPrompt` has no cause, but `checkLivenessTick` already set `run.state.Started`, and its last startup capture is the ready screen from before the death, so it gets no tail.
  - Every exit with a cause gets nothing more.
    A busy, in-draft or unlanded send cause already ends with the pane's last lines through `withPaneTail` or `awaitIdleSession`.
    The remaining causes report a mechanism failure with no screen: the reed-status retry exits, the skill-loader lookup, the prompt-offset persist, and a raw key or paste delivery error.
    So no error gets a second tail.
  - A run with no successful startup capture gets no tail, and its error still says no pane capture was saved.
  Update `abandonStartup`'s doc comment to state when the error ends with the tail.

Tests:
- `TestStartup_PaneNotLiveMidStartup`: `lastCapture` becomes 17 non-blank lines (`line 1` to `line 17`) with one blank line among them.
  The `startup-capture.txt` assertion is unchanged, so the file still holds the whole capture.
  Every row asserts that the error ends with `The pane's last lines:\n` followed by `line 3` to `line 17` joined by newlines.
  That pins the never-ready tail, the cut to `paneTailLines` and the dropped blank line.
- `TestStartup_CaptureAlwaysErroringUntilWindowExpires` asserts that the error does not contain `The pane's last lines`, since no capture ever succeeded.
- `TestStartup_SkillLoadTurns`:
  - `skillReed` gains a `Screen string` field, and `skillReed.CapturePane` writes it ahead of the typed texts, so a test can give the pane a screen before anything is typed.
  - The table gains a `screen` field, which the loop's `configure` copies into `reed.Screen`.
  - The "pane that dies during a load turn is a died startup" row sets `screen` to a ready screen, so `lastStartupCapture` is not empty when the pane dies after ready.
  - Rows with `wantDied` assert that the error does not contain `The pane's last lines`.
    An implementation that keyed the tail on a nil cause alone would add one there.
- `TestStartup_MechanismFailure`:
  - The table gains a third row, "reed stops tracking the strand after a capture".
    Its inner `fakeReed` has a `StatusQueue` that lists `strand-1` live once and then only `someone-elses-strand`, and a `CaptureQueue` of `still booting`.
    The first probe captures `still booting` and reads pending.
    The next two probes return `errStrandNotTracked`, so the retry cap ends the start with a cause while `lastStartupCapture` is not empty.
    The row's `wantErr` is `errStrandNotTracked`.
  - Every row asserts that the error does not contain `The pane's last lines`, which pins that an exit with a cause gets no tail.

**ImpactSummary:** `paneTail`'s cut moves into `lastPaneLines` with unchanged output for every send-failure caller. `abandonStartup` gains a tail on its never-ready, no-cause exits only, and its callers in `awaitStartup` and `loadSkillsThenPrompt` and the `ErrNotStarted` wrap are unchanged.

**Commit:** 3: A never-ready shuttle start's error ends with the pane's last lines
