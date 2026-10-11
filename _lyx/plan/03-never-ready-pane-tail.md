# Card 3 — A never-ready teardown's error ends with the pane's last lines

**Edit:**
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_NeverReady`
- `internal/shuttleengine#TestStartup_MechanismFailure`
- `internal/shuttleengine#TestStartup_CaptureAlwaysErroringUntilWindowExpires`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine#auditedNegativeVerdictReturns`

**Create:**
- `plan:internal/shuttleengine#captureTail` -> `func captureTail(capture string) string`
- `plan:internal/shuttleengine#Run.abandonNeverReady` -> `func (run *Run) abandonNeverReady(outcome Outcome) (Result, error)`

**Intent:** Decision D4, as the parent's addendum states its scope: never-ready with no cause.

`captureTail` (in `send.go`) returns the last `paneTailLines` non-blank lines of a capture string, each right-trimmed, joined by newlines.
`paneTail` keeps its behavior and cuts its successful capture through `captureTail`.

`Run.abandonNeverReady` (in `wait.go`) calls `run.abandonStartup(outcome, nil)` and, when `run.lastStartupCapture` holds a non-blank line, returns the error extended as `<err>. The pane's last lines:\n<captureTail(lastStartupCapture)>`, still wrapping the original error so `errors.Is(err, ErrNotStarted)` holds.
With no capture, or a blank one, it returns `abandonStartup`'s result and error unchanged.
`Run.awaitStartup`'s three exits where the provider never reached `StartupReady` and no cause exists call `abandonNeverReady` instead of `abandonStartup(…, nil)`: the died answer from `checkLivenessTick` (a dead pane or an expired startup window), the run-deadline `OutcomeTimeout` exit, and the tick-cap exit.
The reed-status retry exits keep `abandonStartup` with their cause and get no tail.
`loadSkillsThenPrompt` is unchanged: its died skill-load turn keeps `abandonStartup(OutcomeDied, nil)`, since its last startup capture is the ready screen from before the death, and its cause exits keep theirs.
A busy, draft or unlanded send cause already ends with the pane's last lines through `withPaneTail`; the other causes report a mechanism failure with no screen; none gets a second tail.
`abandonStartup`'s doc comment states that a nil cause also comes from the skill-load death and that the tail is `abandonNeverReady`'s; `awaitStartup`'s return-mapping paragraph names the tail.

`abandonNeverReady` sits in `wait.go`, which the completion-signal tripwire scans, and its extended `fmt.Errorf` return is a new negative-verdict return site.
`auditedNegativeVerdictReturns` gains `"abandonNeverReady [Errorf]": 1`, and its audit comment gains the entry: the tail-extending return of the never-ready teardown, reached only from `awaitStartup`'s three guarded not-ready exits, after `abandonStartup` already finalized.
The three `awaitStartup` exits keep their `OutcomeDied` or `OutcomeTimeout` argument, so their ledger counts are unchanged.

Tests.
`TestStartup_TickCap`'s `NoOutputFiles_ErrNotStartedWithTeardown` subtest scripts one capture of `paneTailLines`+3 lines, `line 1` to `line <paneTailLines+2>` with one blank line among them, and asserts the error ends with `The pane's last lines:\n` followed by the last `paneTailLines` non-blank lines joined by newlines, which pins the tick-cap exit's tail and the cut.
`TestStartup_PaneNotLiveMidStartup` asserts on every row that the error ends with `The pane's last lines:\nstill booting...`.
`TestStartup_RunDeadlineShorterThanWindow_NeverReady`'s rig scripts a `booting` capture, and its `StartGated` subtest asserts the timeout teardown's error ends with `The pane's last lines:\nbooting`.
`TestStartup_CaptureAlwaysErroringUntilWindowExpires` asserts its error carries no `The pane's last lines`.
`TestStartup_MechanismFailure` gains the row "reed stops tracking the strand after a capture": status queue live `strand-1` then `someone-elses-strand`, capture queue `booting`, wanting `errStrandNotTracked`; every row asserts the error carries no `The pane's last lines`.
`skillReed` gains a `Screen string` field that `skillReed.CapturePane` writes ahead of the typed texts, so the startup probe records a non-blank capture.
`TestStartup_SkillLoadTurns` gains a `screen` field that its configure closure copies onto the reed; the "pane that dies during a load turn is a died startup" row sets it to `ready screen`, and a died row asserts the error contains neither `The pane's last lines` nor `ready screen`.

**ImpactSummary:** Three not-ready exits of `awaitStartup` extend their error text; `paneTail`'s output is unchanged, and the cause-carrying exits and every send failure keep their current text.

**Commit:** 3: End a never-ready start's error with the pane's last lines
