# Plan-Review round 2 — fixer report

All 32 findings were checked against the code and the decision record; every premise held, and every finding is fixed.
`lyx loom validate-plan` over the fixed plan reports `ok: true` with one informational finding (`create-new-unit` for card 18's new `roleguard_test.go`).

## Renumbering

F25 split `FabricReaderVerbs` into its own card, which had to come before the classifier card that uses it, so old cards 13–25 are now 14–26.
New card 13 is `13-fabric-reader-verbs.md`.
Every card file, heading, Card Index line, cross-card reference ("card N") and the `classifier-by-injection` shared decision's `Applies to:` were updated to match.
Below, cards are named by their new numbers.

## Changed

- **F1 (card 21):** `recoverbatch_test.go` joins `Edit:`. `TestRecoverBatch_SecondCall`'s transcript-shell row now expects the strand to keep running, and its name, comment and the test's doc comment are updated. The new `TestTurnEndedAfter_OutstandingWork` and its budget bump are gone: the existing rows already cover a payload shell, a plain turn end and a later turn start over the signal-parsing engine. `strand_test.go` and `test-budget.yaml` leave the card.
- **F2 (card 22):** `Run` gains a gated-arrival mark. `Wait` sets it at every gated Done arrival before it consults `boundaryIdle`. `expiredTurnEnd`'s at-once exception reads that mark and the started-fresh fact, never `gateFails`, `gateSent` or `gatePending`. A row covers a failing arrival whose re-prompt send failed busy. `Run.Wait` joins `Edit:`.
- **F3 (card 22):** `finalize` logs that the strand removal ended the shells only beside a successful `RemoveStrand` on its cleanup branch. Every other end (non-Done, `KeepPane`, failed removal) logs that the run ended with those shells outstanding. The test rows are aligned to match.
- **F4 (card 25):** `//contracts/stencils/claims_test.go` joins `Edit:`. The card names the two pinned strings (`OR` + `status: dead`, and `dead` → `its recovery already failed`), moves them to the new wording, and adds claims for the `recovery_retry` and `recovery_exhausted` rungs.
- **F5 (card 4):** `finishMasterDone` carries the re-sign arrow with a `rebaselineWarning string` parameter. The Intent says `Run` passes it, and `ImpactSummary` names `Run` as the one caller.
- **F6 (card 15):** Each pass-through is now pinned:
  - `auditTerminalFork`: a case in `TestRecordBatch_ForgedTerminalRecordFails`.
  - `runExitAuditCrossCheck`: a row in `TestRun_RunExitRefusals`.
  - `recordBatchCmd`'s supply: a `TestRecordBatchCmd_Envelope` row with a read-only command in the fake audit's `BashCommands`.
  - `runDeps().ReadOnly`: asserted in `TestWire_StandaloneMode`.
  - recover-batch's supply: card 16's envelope row.

  `runlevel_test.go`, webstercli `verbs_test.go` and `wiring_test.go` join `Edit:`.
- **F7 (card 14):** Adds the `||` rows (`ls a || cat b` accepted, `ls a || git push` rejected), one accepted row per `lyx fabric` reader, and rejected rows for `prune --apply`, `cleanup --apply` and `shortname x`.
- **F8 (card 20):** `reset_test.go` joins `Edit:`. `TestResetCmd` gains a row forcing the unknown-parent refusal and asserting the clause. Every substring-matched test now asserts the clause in one of its assertions.
- **F9 (card 3):** The three cause rows with their captured Info line live in `TestRebaseline_OverviewFrameUnreadable`. That test drops `t.Parallel` for itself and its rows, under a comment naming the logger's process-global output, and its doc comment is widened.
- **F10 (card 3):** Adds a step-set row for the overview-read transient. The restamp wrap now applies only with a step set; with an empty `Step` the `restampBaseline` text stays as today's, so the "empty Step keeps today's texts" claim holds.
- **F11 (card 22):** `Run.Wait` is in `Edit:`, and its held-turn-end log drops the outstanding count. `noticeMaxTasks` is listed under `Delete:`.
- **F12 (card 22):** `expiredTurnEnd` keeps the counted turn end's shells on `Run` before it clears the waiting list. `pollEventsTick` drops them when it reads a newer turn end. `finalize` falls back to them, so the gated at-once finish records its shells without counting the same turn end twice. A gated at-once row is added.
- **F13 (card 22):** One marker rule, matching D9: the marker is on only once a shell has been outstanding past the bound, and the `waitmark_test.go` rows are aligned to it. `logPayloadShellPastBound` moves from Info to Warn, as D9 states.
- **F14 (card 22):** `internal/shuttleengine/template.yaml` and `waitshadow_test.go` join `Edit:`. The template comment is reworded, and the expired-shell row is renamed and reworded.
- **F15 (card 16):** `AcceptBatchFabricReference` now takes `verb string`, which words both the refusal's re-run and each recorded warning's `accepted by <verb>` detail. Rows assert the detail.
- **F16 (card 16):** Adds a mixed-batch row and a path-carrying fabric-reference row, each refusing `needs_fresh`.
- **F17 (card 11):** The Intent states the new `internal/pattern` import, its transitive tree, and that no cycle results. The `doc.go` leaf and `lyxcwd` sentences are reworded to what still holds.
- **F18 (card 11):** Adds rows for a fence with an info string, an unclosed fence, and the second of two fenced blocks.
- **F19 (cards 7, 11):** Each adds a `TestGateParity_PlanGate` fixture (`DoneCardEdited`, `PatternEntryLineCap`) and lists `internal/loomcli/parity_test.go`.
- **F20 (card 8):** An unreadable record now renders a fixed note saying the frozen cards are unknown and the plan gate reports the read error, never `none`. A row pins it.
- **F21 (card 6):** Drops the edited-done-card row from `TestEditedDoneCards`; card 7's `TestNewPlanGate` covers it.
- **F22 (cards 6, 7):** Adds a recorded-id-the-plan-lacks row to card 6 and an unreadable-done-card error row to card 7.
- **F23 (card 20):** The false "no test reads it" premise is removed. The clause is pinned in `hubgeom/geometry_integration_test.go`'s existing row, so the new `TestReadOriginFor`, `origin_test.go` and the budget bump are dropped.
- **F24 (card 25):** `resetCmd`'s long help names the batch-start count clear.
- **F25 (new card 13):** `FabricReaderVerbs` has its own `fabric-reader-verbs` card. Card 14 lists it under `Uses:` and card 18 still does.
- **F26 (card 17):** Each branch is named against the existing `TestGateTest_Scenario` subtest that drives it (free slot, inherited held slot, outside every hub), and each asserts the variable absent. The `Verify` row names the nil-env unslotted run.
- **F27 (card 18):** `roleguard_test.go` holds one top-level test, `TestSandboxedRoleGuard`. The `lyx config` row moves into `TestConfigOverRealHub`'s per-worktree subtest, whose sync is the real `RunCLIIn`, and `guardRows` is untouched.
- **F28 (cards 4, 24):** `TestRun_AutoRebaseline` gains three rows: `Fresh` beside `AutoRebaseline`, an error after an accepted rebaseline opening with the warning while still matching `ErrPendingAuditFindings`, and the captured Warn line. `TestRecoveryRetry`'s failed-read row asserts its Warn line. Both tests run non-parallel under a comment naming the logger's output.
- **F29 (card 4):** After an accepted rebaseline, `Run`'s plan-validation refusal, zero-batch refusal and quarry transient name the re-step and are wrapped in `ErrAutoRebaseline`, so the adapter maps them to Stuck. `zeroBatchesError` is re-signed with a `step` parameter (an empty step keeps today's text). A validation-refusal row is added.
- **F30 (card 21):** `TestRecoverBatch_SecondCall` gains a row with a shell outstanding and the report present, which classifies done from the report.
- **F31 (card 1):** `TestRun`'s table gains a `stdin` field and a `viaStdin` dispatch flag. The stdout-kept case moves to `TestRun_StdoutOnError`.
- **F32 (card 23):** Adds an interleaved ordering row: `TestA/s1`, `TestB` and `TestA/s2` in one package, plus `TestA/s1` in a second package.

## Constraint over fasit

F1 follows `PATTERN-test-economy` over a sentence in D9 of the decision record.
That sentence says `TurnEndedAfter`'s outstanding-shell cases "need a new test over a signal-parsing fake engine".
Its premise was that `TestTurnEnded` cannot reach those cases, which is true, but `TestRecoverBatch_SecondCall` already drives `TurnEndedAfter` through `RecoverBatch` with `claudeengine`, the signal-parsing engine, for both shell signals.
A new top-level test would therefore duplicate covered behavior.
The instruction says a constraint wins over a conflicting text, so card 21 extends the existing test instead.

## Disputed

None.

## Deferred

None.
