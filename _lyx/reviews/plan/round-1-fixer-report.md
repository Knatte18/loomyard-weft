# Plan-Burler round 1 — fixer report

Every finding was checked against the code and fixed.
`lyx loom validate-plan` over the edited plan reports `ok: true`, with only the two informational `create-new-unit` findings card 9 already carried.

## Fixed

- **F1** (card 1 misses `run_resume_test.go`): confirmed at `internal/shuttleengine/run_resume_test.go:85` and `:121`.
  Card 1 lists `internal/shuttleengine/run_resume_test.go` under `**Edit:**`, and its Intent drops the two `startedFresh` assertions and keeps the `ResumeWarning` and `CheckResume` assertions.
- **F2** (tick-level table on the old rule): confirmed at `internal/shuttleengine/waiting_test.go:201-268`.
  Card 1 now reworks `TestPollEventsTick_OutstandingBackgroundWork`: it flips the two fresh-gated rows to expect waiting, drops the `fresh` and `arrived` columns and the two rows that then duplicate the flipped transcript row, and rewords the doc comment and `//testtiming:keep` line.
- **F3** (redundant reworked hold test): confirmed against `TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival` (`waiting_test.go:126`) and the awaited-shell row of `TestWait_RecordsEndedShells`.
  Card 1 now deletes `TestWait_GatedFreshRunWithOutstandingShellEvaluatesGateAtOnce`.
  It drops `TestWait_RecordsEndedShells`' `fresh` column and adds a `wantGateCalls` column, an unawaited-shell `STOP` row, and the deadline row the card already planned.
- **F4** (`NewVerifyGate` error path): confirmed that `NewVerifyGate` (`internal/websterengine/verifygate.go:125`) has no error result, and that `Run` resolves `verifyFixPromptPath` at `runlevel.go:678`, before it builds the gate at `:719`.
  Card 4 drops the `verifyFixPromptFile` Create.
  `NewVerifyGate` takes a `fixPromptPath` parameter, marked with the re-sign arrow, and `Run` passes the path it already resolves, so no new error path arises.
  `internal/websterengine/verifygate_integration_test.go` and `internal/webstercli/verify_parity_integration_test.go` are listed under `**Edit:**`, the test paragraph updates their calls, and the ImpactSummary counts the new re-sign.
- **F5** (bound on the weakened guard unstated): card 10's Intent now states D6's bound.
  Only candidates that provably never coexist pass.
  The cap and every read or parse failure fall back to `ambiguous`, and no tag is held true by the building toolchain.
  A Rename's old side stays one declaration, and the Delete direction keeps blocking.
  Cards 11 and 12 each restate the bound for their own guard and point at card 10.

## Disputed

None.

## Deferred

None.
