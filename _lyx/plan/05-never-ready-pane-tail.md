# Card 5 — A never-ready teardown's error ends with the pane's last lines

**Create:**
- `plan:internal/shuttleengine#captureTail` -> `func captureTail(capture string) string`
- `plan:internal/shuttleengine#Run.abandonNeverReady` -> `func (run *Run) abandonNeverReady(outcome Outcome) (Result, error)`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_NeverReady`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine/doc.go`

**Intent:** D4, with the parent's scope amendment: the tail goes on never-ready exits that carry no cause, and on nothing else.
`captureTail` is the cut `paneTail` already does, pulled out as a pure function over a capture string: the last `paneTailLines` non-blank lines, right-trimmed, joined by newlines.
`paneTail` keeps its signature and its capture-failed sentence and delegates the cut to `captureTail`, so send failures read as before.
`abandonNeverReady` calls `abandonStartup(outcome, nil)` and, when `run.lastStartupCapture` is non-empty and `captureTail` of it is non-empty, returns the result with the error wrapped as `fmt.Errorf("%w. The pane's last lines:\n%s", err, tail)`, so the returned error still wraps `ErrNotStarted` and ends with the tail; with no capture it returns `abandonStartup`'s answer untouched.
`abandonNeverReady` lives in `wait.go` beside `abandonStartup` and builds the extended error by assignment (`err = fmt.Errorf(...)`) and then `return result, err`, never a `return` statement carrying `fmt.Errorf` or an `Outcome*` identifier, so `TestCompletionSignal_NegativeVerdictReturnSites` (`completionsignal_enforcement_test.go`, scanning `wait.go`) counts no new negative-verdict return for it; `awaitStartup`'s own returns keep the `OutcomeDied` and `OutcomeTimeout` arguments, so the audited `awaitStartup [OutcomeDied]` x2 and `[OutcomeTimeout]` x1 counts hold and `auditedNegativeVerdictReturns` needs no edit.
`awaitStartup` calls it at exactly its three never-ready, no-cause exits: the pane died (`outcome == OutcomeDied`), the run-deadline expiry (`OutcomeTimeout`) and the tick-cap exit.
The three reed-status retry exits keep calling `abandonStartup` with their cause and get no tail, and `start`'s callers of `abandonStartup` in `run.go` are untouched.
That leaves the skill-load death, which calls `abandonStartup(OutcomeDied, nil)` from `loadSkillsThenPrompt`, tail-free: its last startup capture is the ready screen from before the death.
A cause that is present adds nothing: the busy, in-draft and unlanded send causes already end with the pane's last lines through `withPaneTail`, and the skill-loader lookup, the prompt-offset persist and a raw key or paste delivery error report a mechanism failure with no screen.
What it lets through: nothing new is skipped; it appends text to an error that is already returned, from a capture already held in memory (no extra pane capture and no new reed call).
The `Result` of the three exits is unchanged, and `RunGated` carries the extended text on `Result.StartError` through card 3.

Tests extend existing tables; no new top-level test.
- `TestStartup_PaneNotLiveMidStartup` gains a `wantTail` assertion on its pane-goes-not-live row: the error ends with the last capture (`still booting...`), and a new row with a capture of more than `paneTailLines` numbered non-blank lines interleaved with blank lines asserts the error ends with exactly the last `paneTailLines` of them.
  Both rows' `startup-capture.txt` assertions stay.
- `TestStartup_TickCap`'s `NoOutputFiles_ErrNotStartedWithTeardown` subtest reaches the tick-cap exit: its fake reed has no capture queued, so the capture is empty and the error must not contain `The pane's last lines:` (the "a startup capture exists" condition); a second subtest with a one-entry `CaptureQueue` asserts the error ends with that capture.
- `TestStartup_RunDeadlineShorterThanWindow_NeverReady` reaches the run-deadline exit: it queues a capture and asserts the `OutcomeTimeout` error ends with it.
- `TestStartup_MechanismFailure` is not extended: its status errors fail before any capture is taken (`flakyStatusReed` fails the first calls, `startup_test.go:49-55`), so a no-tail assertion there would be vacuous; the absence of a tail on the retry exits is structural, as only `abandonNeverReady` appends one.
- `TestStartup_SkillLoadTurns` asserts, on its `wantDied` row ("pane that dies during a load turn is a died startup"), that the error text does not contain `The pane's last lines:`.
- `TestStartup_SendFailuresTearDown` gains an explicit `wantTails` count on every row: each row whose cause is a busy, in-draft or unlanded send (every row but the prompt-offset-persist row and the late-landed row, which returns a run) asserts `strings.Count(err.Error(), "The pane's last lines:") == 1`, and the prompt-offset-persist row asserts 0, pinning that a present cause never gets a second tail.
  The skill-load death row's pane is a ready screen with a non-empty capture, which the row asserts through `run`'s `startup-capture.txt` being non-empty, so its no-tail assertion is not vacuous.
`TestStartup_UndismissableGateUntilWindowExpires` and `TestStartup_TickCap` keep passing unchanged; their fakes capture an empty or gate screen and assert only `ErrNotStarted`.
`internal/shuttleengine/doc.go`'s paragraph on the not-ready teardown gains one sentence: a never-ready start's error ends with the last non-blank lines of its last startup capture, and `Result.StartError` carries that text for a single-LLM producer.

**ImpactSummary:** `awaitStartup`'s three no-cause exits return a longer error that still wraps `ErrNotStarted`; `paneTail`'s output is byte-identical, and the reed-status exits and every `run.go` teardown are unchanged.
