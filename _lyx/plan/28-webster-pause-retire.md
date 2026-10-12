# Card 28 — webster-pause-retire

**Delete:**
- `internal/websterengine/pause.go`
- `internal/websterengine/pause_test.go`
- `internal/webstercli/pause.go`
- `internal/websterengine#ErrPaused`
- `internal/websterengine#outcomePaused`
- `internal/shedadapters#websterOutcomePaused`
- `internal/websterengine_test#TestRun_PausedOutcomeLeavesPauseFlagIntact`
- `internal/webstercli#TestBeginBatchCmd_PausedEnvelope`
- `internal/webstercli#TestPauseCmd_ResolvesSameFileAsBeginBatchGate`
- `internal/webstercli#TestPauseCmd_RequestsPauseIdempotent`
- `internal/shedadapters#TestWebsterProducer_OutcomePaused`

**Edit:**
- `internal/websterengine#BeginBatch`
- `internal/websterengine#Run`
- `internal/websterengine#mapMasterDone`
- `internal/websterengine#parseOutcome`
- `internal/websterengine#finishedShellOutcome`
- `internal/webstercli#websterCLI.addVerbs`
- `internal/webstercli#websterCLI.statusCmd`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `internal/shedadapters#WebsterProducer.Call`
- `internal/websterengine_test#TestBeginBatch_Refusals`
- `internal/websterengine_test#TestRunInFlight`
- `internal/websterengine#TestParseOutcome_AcceptReject`
- `internal/websterengine_test#TestRun_DoneOutcome`
- `internal/websterengine_test#TestRun_ExpiredShells`
- `internal/websterengine_test#TestRun_RefusesBeforeSpawn`
- `internal/websterengine#TestBackgroundShellNoteAndWarning`
- `internal/websterengine#TestVerifyGate_PassesWithoutVerifying`
- `internal/webstercli#TestBringUpDisposition_RecoverAndRun`
- `internal/webstercli#TestNoteRefusals_VerbEnvelopes`
- `internal/webstercli#TestBeginBatchCmd_DeleteTargetAlreadyGone`
- `internal/webstercli#newPairFixtureAt`
- `internal/webstercli#TestFabricSync_CommitsAtEveryRelPathDepth`
- `internal/shedadapters#TestWebsterProducer_CancelledDuringRun_StuckPausedAndErrorBecomeContextError`
- `internal/shedadapters#TestWebsterProducer_NoBridgeInstalled`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/claims_test.go`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/doc.go`
- `internal/shedadapters/doc.go`

**Intent:**
Retire `lyx webster pause` and webster's pause flag (D8), whose use `lyx pause` (to stop the agents) and `lyx loom pause --after` (to halt at a boundary) now cover.

- `websterengine/pause.go` (`RequestPause`, `PauseRequested`, `ClearPause`, `PauseFlagName`, `pauseFlagPath`) and its test file go; `BeginBatch` drops its pause-flag check and `ErrPaused`; `Run` and `mapMasterDone` drop their `ClearPause` calls and `mapMasterDone` its paused branch.
- Webster's `paused` outcome goes: `parseOutcome` accepts `done` and `stuck` only and names both in its refusal, `finishedShellOutcome` drops its paused case, and `shedadapters`' `WebsterProducer.Call` drops `websterOutcomePaused`, so an outcome file reading `paused` is the malformed-outcome refusal its row already names.
- webstercli drops `pause.go`'s `pauseCmd` and its registration in `addVerbs`, `beginBatchCmd`'s `{"paused": true}` envelope and the help sentence naming it, and `statusCmd`'s `paused` key.
- The Master stencil drops "A paused refusal ends your run immediately" and the `paused` outcome value, and `claims_test.go`'s claims pinning them go with them.
- `refusal-spec.md`'s webster row `Webster row paused out of band` goes.
- The docs name the outcomes `done` and `stuck` only.

Tests: the deleted tests pinned only the retired flag, verb, envelope or outcome; each edited test drops its paused row or assertion (`TestBeginBatch_Refusals`' flag row, `TestParseOutcome_AcceptReject`'s accepted paused value, which now refuses, the paused cases of the shell, run-level, verify-gate, in-flight, bring-up, refusal-note and producer tests), keeping its other rows unchanged; the records-sync fixture `newPairFixtureAt` and `TestFabricSync_CommitsAtEveryRelPathDepth` keep a machine-local webster artifact in the held-back set through a literal file name in place of the retired `PauseFlagName`.

**ImpactSummary:** Deletes webster's pause verb, flag and outcome value across websterengine, webstercli, shedadapters, the Master stencil and their tests; no other module set or read the flag.
