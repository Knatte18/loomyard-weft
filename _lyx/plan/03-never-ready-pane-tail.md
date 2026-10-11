# Card 3 — never-ready-pane-tail

**Edit:**
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`

**Create:**
- `plan:internal/shuttleengine#lastPaneLines` -> `func lastPaneLines(capture string) string`

**Uses:**
- `internal/shuttleengine#paneTailLines`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`

**Intent:**
A never-ready start's error ends with the pane's last lines (decision D4 and the parent's addition stating its scope), so with card 2 the producer message shows the screen that ended the start without opening the trace or the saved capture file.

`lastPaneLines` returns the last `paneTailLines` non-blank lines of capture, each right-trimmed, joined by newlines: the cutting `paneTail` does today.
`paneTail` keeps its capture and its failure sentence and calls `lastPaneLines` for the cut, so send failures keep their exact text.

`Run.abandonStartup` appends `. The pane's last lines:\n<lastPaneLines(run.lastStartupCapture)>` to its returned error when, and only when, `cause` is nil, `run.state.Started` is false and `run.lastStartupCapture` is non-empty.
That condition is the never-ready-with-no-cause trigger: `awaitStartup`'s three nil-cause exits (the pane died before ready, the run deadline, the tick cap) all run before `checkLivenessTick` ever set `Started`.
The reed-status retry exits carry a cause and get no tail.
The skill-load death in `loadSkillsThenPrompt` passes a nil cause but runs after ready, with `Started` true, so its last startup capture is the ready screen from before the death and it gets no tail.
A cause-carrying teardown gets nothing added: a busy, in-draft or unlanded send cause already ends with the pane's last lines, and the remaining causes report a mechanism failure with no screen.
The error still wraps `ErrNotStarted`, and `abandonStartup` keeps its single return statement, which the audited-return scan in `completionsignal_enforcement_test.go` counts as one.

Tests:
- `TestStartup_PaneNotLiveMidStartup` asserts the error ends with `"The pane's last lines:\n" + lastCapture`.
- `TestStartup_SkillLoadTurns`'s row "pane that dies during a load turn is a died startup" asserts the start error does not contain `The pane's last lines`.

**ImpactSummary:** `abandonStartup`'s error text gains a trailing tail on its never-ready nil-cause path only; `paneTail` is refactored with unchanged output for every send-failure caller.

**Commit:** 3: A never-ready start's error ends with the pane's last lines
