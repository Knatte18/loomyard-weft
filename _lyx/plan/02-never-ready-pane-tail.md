# Card 2 — A never-ready start's error ends with the pane's last lines

**Create:**
- `plan:internal/shuttleengine#captureTail` -> `func captureTail(capture string) string`
- `plan:internal/shuttleengine#Run.abandonNeverReady` -> `func (run *Run) abandonNeverReady(outcome Outcome) (Result, error)`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#auditedNegativeVerdictReturns`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_CaptureAlwaysErroringUntilWindowExpires`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_NeverReady`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`

**Intent:** Decision D4: on `awaitStartup`'s not-ready exits, where the provider never reached `StartupReady` and `abandonStartup` gets no cause, and a startup capture exists, the returned error ends with the last non-blank lines of that capture, cut to `paneTailLines`.

`captureTail` (in `send.go`) takes `paneTail`'s line-cutting body: the last `paneTailLines` non-blank lines of `capture`, each right-trimmed, joined by newlines.
`paneTail` keeps its signature and its capture-failure sentence and returns `captureTail(capture)` after a successful capture.

`abandonNeverReady` (in `wait.go`) calls `run.abandonStartup(outcome, nil)` and, when `run.lastStartupCapture` is non-empty, returns the same `Result` with the error extended as `fmt.Errorf("%w. The pane's last lines:\n%s", err, captureTail(run.lastStartupCapture))`, the wording a send failure's tail uses, so `errors.Is(err, ErrNotStarted)` still holds; with no capture it returns `abandonStartup`'s answer unchanged.
`awaitStartup`'s three cause-less exits call it instead of `abandonStartup(…, nil)`: the not-ready answer (`OutcomeDied` from `checkLivenessTick`, a dead pane or an expired window), the run-deadline exit (`OutcomeTimeout`) and the tick-cap fallback (`OutcomeDied`).
The three reed-status retry-cap exits keep `abandonStartup` with their cause and get no tail; so do `loadSkillsThenPrompt`'s exits, the pane that dies during the skill-load turn included: it is not never-ready, and its last startup capture is the ready screen from before the death.
`abandonStartup` itself is unchanged.

`completionsignal_enforcement_test.go`'s `auditedNegativeVerdictReturns` gains `"abandonNeverReady [Errorf]": 1`, with a comment line beside the `abandonStartup` one: its one `Errorf` return extends `abandonStartup`'s ErrNotStarted error with the capture's tail and is reached only from `awaitStartup`'s three guarded not-ready exits.
The `awaitStartup` counts are unchanged, since each return still names only `OutcomeDied` or `OutcomeTimeout`.

`doc.go`'s closing startup paragraph adds that a never-ready teardown's error ends with the last lines of the last startup capture.

Tests:
- `TestStartup_PaneNotLiveMidStartup` (the not-ready exit): the rows gain a `capture` field replacing the shared `lastCapture` constant, and every row asserts `err.Error()` ends with `"The pane's last lines:\n"` plus the expected tail, the strand-removal-failure row included.
  A new row's capture holds `paneTailLines + 5` numbered non-blank lines with blank lines between them, and its error ends with exactly the last `paneTailLines` of them joined by newlines; the existing rows keep `still booting...` as the whole tail.
- `TestStartup_CaptureAlwaysErroringUntilWindowExpires`: the error, with no capture ever taken, does not contain `The pane's last lines`.
- `TestStartup_TickCap`'s `NoOutputFiles_ErrNotStartedWithTeardown` subtest (the tick-cap exit) seeds `CaptureQueue: []string{"still booting..."}` and asserts the error ends with `"The pane's last lines:\nstill booting..."`.
- `TestStartup_RunDeadlineShorterThanWindow_NeverReady`'s `StartGated` subtest (the run-deadline exit) seeds the same capture and asserts the same ending.
- `TestStartup_SkillLoadTurns` (the skill-load death, which must get no tail): `skillReed`'s capture echoes only typed text, so the startup probe's capture is empty today and the row could not tell a tail from none.
  `skillReed` gains a `Screen string` field, the pane's text before anything is typed, and `skillReed.CapturePane` writes it, followed by a newline when non-empty, ahead of the echoed texts; `typedTexts` reads `SendTextCalls` and is unaffected.
  The rows gain a `screen` field the fixture's `configure` callback copies into `reed.Screen`; the `pane that dies during a load turn is a died startup` row sets it to `ready screen`, so the startup capture is non-empty, and asserts the error does not contain `The pane's last lines`.
  With the tail wrongly added to every cause-less `abandonStartup`, this row fails.

**ImpactSummary:** `paneTail`'s output is unchanged; only `awaitStartup`'s three cause-less teardowns change their error text, by an appended tail, with `ErrNotStarted` still wrapped; the completion-signal tripwire gains one audited site.

**Commit:** 2: A never-ready start's error ends with the pane's last lines
