# Card 25 — driftengine-retire-acceptance

**Delete:**
- `internal/driftengine#testFiles`
- `internal/driftengine#requireBaselineTest`
- `internal/driftengine#noBaselineTestRefusal`
- `internal/driftengine#keepAcceptedOutput`
- `internal/driftengine#dropAcceptedOutput`
- `internal/driftengine#buildAcceptedProfile`
- `internal/driftengine#RecordScope`
- `internal/driftengine#dropCaptureSkipping`
- `internal/driftengine#Accept`
- `internal/driftengine#CheckAccept`
- `internal/driftengine#acceptingCaller`
- `internal/driftengine#admitAcceptance`
- `internal/driftengine#Withdraw`
- `internal/driftengine#Waive`
- `internal/driftengine#RunCount`
- `internal/driftengine#CheckVerifyRow`
- `internal/driftengine#VerbStatus.AtAgentSession`
- `internal/driftengine#VerbStatus.AtOperatorSession`
- `internal/driftengine#VerbStatus.AtDecisionHalt`
- `internal/driftengine#ScopeRecord.Unruled`
- `internal/driftengine#runsRecord`
- `internal/driftengine#ErrRunsSpent`
- `internal/driftengine#updateRuns`
- `internal/driftengine#readRuns`
- `internal/driftengine#TakeRun`
- `internal/driftengine#ResetRuns`
- `internal/driftengine#AttemptHead`
- `internal/driftengine#CloseAgentSession`
- `internal/driftengine#AgentSessionOpen`
- `internal/driftengine#OperatorSessionOpen`
- `internal/driftengine#SideResult`
- `internal/driftengine#RunSideCapture`
- `internal/driftengine#AcceptanceCapture`
- `internal/driftengine#RunAcceptanceCapture`
- `internal/driftengine#captureInto`
- `internal/driftengine#siteSideCapture`
- `internal/driftengine#siteAcceptanceCapture`
- `internal/driftengine#SkipList`
- `internal/driftengine#TestRecordScope`
- `internal/driftengine#TestAcceptWithdraw`
- `internal/driftengine#TestTakeRun`
- `internal/driftengine#writeProfileFile`
- `internal/driftengine#stageAcceptance`
- `internal/driftengine#keptOutput`
- `internal/driftengine#verifyFixture.agentAccept`
- `internal/driftengine#verifyFixture.sessionAccept`
- `internal/driftengine/skiplist_test.go`
- `internal/driftcli#TestVerbStatus_OperatorSession`

**Edit:**
- `internal/driftengine#VerbStatus`
- `internal/driftengine#Paths`
- `internal/driftengine#NewPaths`
- `internal/driftengine#missingDirRefusal`
- `internal/driftengine#TestDeferWaive`
- `internal/driftengine#statusAt` -> `func statusAt(row string, state shedengine.State, errText string) VerbStatus`
- `plan:internal/driftengine#TestAmend`
- `internal/driftengine#TestDriftAwait`
- `internal/driftengine#TestDriftBaseline`
- `internal/driftengine#newBaselineFixture`
- `internal/driftengine#TestDriftVerify`
- `internal/driftengine#TestRunCapture`
- `internal/driftengine/doc.go`
- `internal/driftcli#worktree.verbStatus`
- `internal/driftcli/doc.go`

**Uses:**
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Remove the acceptance machinery, the run budget and side capture that the retired verbs used (D14).

- `accepted.go` keeps only `copyFile`, which the judge snapshot uses; its acceptance and profile functions go.
- `verbs.go` loses scope recording, accept, withdraw, waive, the run count and the verify-row check; `VerbStatus` drops `AgentSession` and `OperatorSession` and their methods, and `worktree.verbStatus` stops reading the agent-session mark.
- `records.go` loses `runs.json`'s record and functions, whose session mark lives in `scope.json`'s `expect_session` and `judge_session` since cards 12 and 16, and `ScopeRecord.Unruled`; the old acceptance, compare and waiver fields and the `partial` scope stay readable for the verdict and D3's in-flight rule.
- `capture.go` loses the side and acceptance captures and `captureInto`; `missingDirRefusal` drops their sites; `compare.go` loses `SkipList`.
- `Paths` drops `SideDir`, `AcceptedDir`, `AcceptingDir`, `AcceptLock`, `AcceptedTestsDir` and `Runs`, so nothing names `.lym/drift/`'s `accepting`, `accepted-tests` or `accept.lock`.
- `statusAt` drops its `agentSession` parameter with the field, and every call of it, in `TestDeferWaive`, card 19's `TestAmend`, `TestDriftAwait`, `TestDriftBaseline`, `newBaselineFixture` and `TestDriftVerify`, drops the argument; `TestDeferWaive` drops its waive rows and `TestRunCapture` its side and acceptance rows; the package doc drops the `# Verbs` acceptance text, the run counter and the accepted profile.

**ImpactSummary:** Deletes engine functions whose last callers left in cards 16, 22 and 23; one driftcli wiring method stops reading a removed mark; no other package reads them.
