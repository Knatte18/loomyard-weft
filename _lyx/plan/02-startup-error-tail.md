# Card 2 — A never-ready teardown's error ends with the pane's last lines

**Create:**
- `plan:internal/shuttleengine#tailLines` -> `func tailLines(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#ErrNotStarted`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_MechanismFailure`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.CapturePane`

**Uses:**
- `internal/shuttleengine#paneTailLines`

**Intent:**
D4 of the decision record, including its added scope statement.
`tailLines` is `paneTail`'s cut extracted: the last `paneTailLines` non-blank lines of a capture, trailing spaces trimmed, joined by newlines; `paneTail` captures and calls it, so a send failure's text is unchanged.
`abandonStartup` appends `"\n" + tailLines(run.lastStartupCapture)` to its one returned error when `cause == nil`, `run.state.Started` is false and `run.lastStartupCapture` is non-empty.
`Started` is the discriminator because `checkLivenessTick` persists it the moment the provider reads `StartupReady`: it separates a provider that never became ready, whose `awaitStartup` exits (dead pane, run deadline, tick cap) all reach this one site with a nil cause, from the pane that dies in the skill-load turn, which also passes a nil cause but has `Started` true and so gets no tail.
A call with a cause adds nothing: the reed-status retry exits carry a cause and no screen, the busy and unlanded send causes already end with `withPaneTail`, and the skill-loader lookup, the prompt-offset persist and a raw key or paste delivery error report a mechanism failure with no screen.
The change adds no `return` and no `Errorf` site in `wait.go`'s scanned functions beyond the existing one, so the counts `completionsignal_enforcement_test.go` audits stay as they are.
The `abandonStartup` and `ErrNotStarted` doc comments say the error then ends with the tail.

What this can now skip or let through: nothing is skipped; the error text only grows by a bounded block of at most `paneTailLines` lines of the pane's own capture.

Tests, extending existing ones:
- `TestStartup_PaneNotLiveMidStartup` gains a `capture` field and a `wantTail` field.
  The row "pane goes not-live" asserts the error ends with the last capture `still booting...`.
  The `CaptureQueue` seed, `wantCapture` and the `startup-capture.txt` comparison use the row's `capture` instead of the constant.
  A new row feeds a capture of more than `paneTailLines` non-blank lines with blank lines between, and asserts the error ends with exactly the last `paneTailLines` non-blank lines and no earlier one.
- `TestStartup_MechanismFailure` gains a row where reed answers live on the first probe, so `"booting"` is captured, and then reports a foreign strand until the retry cap fires; a `wantNoTail` field asserts the error carries `errStrandNotTracked` and does not contain `booting`.
  The row is reachable because `checkLivenessTick` records the capture before the second probe's status fails.
- `TestStartup_SkillLoadTurns` gains a `banner` field the fixture's `configure` sets on the reed, and the row "pane that dies during a load turn" sets a non-empty banner and asserts the returned error does not contain it.
  `skillReed` gains `Banner string`, which its `CapturePane` writes ahead of the typed texts; the zero value keeps every other test's capture unchanged.
  Without the banner the startup capture is empty and the row could not tell a tail from none.

**ImpactSummary:** `abandonStartup` has six call sites in `awaitStartup` and five in `loadSkillsThenPrompt`, all inside `shuttleengine`; only the no-cause, never-ready calls with a capture change their text, and `paneTail`'s callers see the same output as before.

**Commit:** 2: shuttle: a never-ready start error ends with the pane's last lines
