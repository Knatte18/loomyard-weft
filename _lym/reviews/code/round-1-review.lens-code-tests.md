```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Per-behaviour list**

| Behaviour | Pinning test | Verdict |
|---|---|---|
| `promptTurnStartedSince` counts only a turn start whose instruction is the prompt line, so an operator's turn or the skill-load turn never counts | `run_test.go` `TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted`: the prompt-own-start, operator-start and zero-offset-over-load-turn rows | pinned |
| `DeliverPendingPrompt` types at most once per handle | The same test, which now calls it a second time after every row and asserts false and an unchanged `typed` | pinned |
| `DeliverPendingPrompt` takes `sendMu` before reading the record | none | no concurrent-call test; the lock move is not behaviourally pinned (NIT) |
| `deliverPromptAtHold` types at a held turn end after an operator's turn, once, with no hold notice | `hold_test.go` `TestWait_HeldTurnEndNotice`, row "after an operator's turn start types the deferred prompt once and sends no notice" | pinned |
| `deliverPromptAtHold` types nothing and still sends the notice when the prompt's own turn started | `hold_test.go`, row "after the prompt's own turn start types nothing and is notified" | pinned |
| `deliverPromptAtHold` skips a run whose pause covers it | none | no `prompt` row sets covered; breaking `run.covered` in `hold.go:131` fails nothing |
| `deliverPromptAtHold` skips when a turn start was read past the held turn end (`run.turnStartRead`) | none | no row |
| A failed delivery at a held turn end is logged, the turn end is held as usual (notice path continues), and the next held turn end retries | none | the `sendFails` field exists in the table, but no `prompt` row sets it |
| `Attach` marks the handle `attached`, and Wait delivers a missing prompt at entry | `attach_test.go` `TestAttach_DeliversTheDeferredPrompt`: zero-offset, set-offset and operator-turn rows | pinned |
| Nothing is typed on attach when the prompt's own turn started | `TestAttach_DeliversTheDeferredPrompt`, prompt-own-start row | pinned |
| Nothing is typed on attach when every output file exists | `TestAttach_DeliversTheDeferredPrompt`, `outputExists` row | pinned |
| A failed delivery on attach is logged and the Wait continues | none | no row |
| A non-attached (`Start`) handle's Wait does not deliver at entry | none direct | only implied by the existing Wait tests with no prompt line |
| `Runner.Open` passes `OpenLaunch` a per-token directory beside the marker | `open_test.go` `TestRunner_Open`, the `openDirs`/`DoneWhen` assertions | pinned |
| `Runner.Open` forwards `Launch.CreelHooks` onto the strand, both ways | `open_test.go` `creelTests` | pinned |
| `Runner.Open` removes the session directory when `AddStrand` fails | none | the fake engine writes no directory and no add-failure row asserts one is removed |
| `Claude.OpenLaunch` with a board directory writes `settings.json` (creel hooks only) and the append-prompt file | `command_test.go` `TestOpenLaunch`, row "a board writes the creel settings and the append prompt", via `checkOpenCreelFiles` | pinned |
| `Claude.OpenLaunch` with a board names both files on the fresh and resume lines, and sets `CreelHooks` | `checkOpenCreelFiles` | pinned |
| `Claude.OpenLaunch` with no board writes nothing, passes no `--settings` or append file, creates no directory, and sets no `CreelHooks` | the existing `TestOpenLaunch` rows, with the new assertions | pinned |
| `Claude.OpenLaunch` refuses a bad permission mode before writing anything | `TestOpenLaunch`, row "a board refusal creates nothing" | pinned |
| `writeOpenFiles` fails on an unwritable directory, or half-writes | none | no row, and no cleanup exists to test |
| `creelOpenStopHookCommand` exits 0 whatever the delivery printed, records nothing, and does nothing when the flag is absent | `hookcmd_integration_test.go` `TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock`, the three "Open" rows | pinned |
| `buildOpenSettings` has the Stop delivery, the idle waiter with `asyncRewake`, and the `PostToolUse` mail hook | `checkOpenCreelFiles` | pinned (it compares against the same builders, so it pins the shape but not the command text; the integration test pins that) |
| The run's launch is unchanged by the `creelStopDelivery`, `creelWaitHookEntry` and `creelFlagDirForHook` refactor | the existing `buildSettings`/`Prepare` tests and the original rows of the Stop-hook test | pinned |
| Corpus replay of a deferred-prompt run supplies the prompt's own turn start | `corpus_test.go` replay | pinned (test support) |
| `TestHoldRecordsWriteLock` takes a free lock with a generous wait | the same test | pinned |

**Findings**

1. MEDIUM, class `design`, `hold.go:131` / `hold_test.go` (`TestWait_HeldTurnEndNotice`): `deliverPromptAtHold` has three guards and a failure path with no failing test. The guards are `run.covered` and `run.turnStartRead`, and the failure path is a send error that falls through to the notice cases and is retried. Only the two success-side rows were added. Evidence: the diff's `hold_test.go` hunks add two rows, and the `sendFails` field exists but no `prompt` row sets it. Removing the `covered` guard, which would type into a paused pane, passes every test.
2. LOW, class `design`, `hold.go:147-152` / `attach_test.go`: the failure branch of `deliverPromptOnAttach` (an error from `DeliverPendingPrompt`) has no row. Evidence: all five rows in `TestAttach_DeliversTheDeferredPrompt` use a working `echoReed`.
3. LOW, class `design`, `open.go:501`: the `os.RemoveAll(sessionDir)` on `AddStrand` failure is uncovered. Evidence: the `open_test.go` additions only cover the `CreelHooks` forwarding and the directory and marker naming, and `openerFakeEngine.OpenLaunch` creates no directory to remove.
4. NIT, class `consistency`, `attach_test.go` rows 3 and 4: the operator-turn and own-turn-start rows re-test the matching logic that `run_test.go`'s `TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted` already pins. The wiring is what `TestAttach_DeliversTheDeferredPrompt` should pin, so the rows are partial overlap. They are cheap tier-0 rows and they test a different path (Wait entry), so they are arguably acceptable.

No finding for `writeOpenFiles` write errors: they are operational errors that propagate unchanged, and the lens does not require a row for each.

Tool calls: none; the inherited context held every diff and file I needed.
