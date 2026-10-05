# Card 1 — Remove Tier 1 GitHub filing from loom

**Edit:**
- `internal/loomcli/selfreport.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/halt.go`
- `internal/loomcli/cli.go`
- `internal/loomengine/anomaly.go`
- `internal/loomengine/config.go`
- `internal/loomengine/anomaly_test.go`
- `internal/loomengine/awaiting_test.go`
- `internal/loomcli/selfreport_test.go`
- `internal/loomcli/halt_test.go`
- `cmd/lyx/pathrules_test.go`

**Delete:**
- `internal/loomengine/anomalybody.go`
- `internal/loomengine/anomalybody_test.go`
- `internal/loomengine/loomstatus_test.go`
- `internal/loomcli/selfreport_github_test.go`

**Intent:**
No `lyx loom run` or `lyx loom step` path files a GitHub issue any more (decision D1).
Delete the whole Go filer outright, with no replacement:

- `internal/loomcli/selfreport.go`: delete `selfreportDeps`, `detectAndFileAnomalies`, `detectFinalAnomalies`, `runFilingPass`, `collapseAnomaliesByTitle`, `pendingAnomaly`, `selfreportFiledMarker` with its methods, `readFiledMarker` and `writeFiledMarker`, and the `selfreportengine` import.
  `observeEntry` and the step-handoff helpers stay; rewrite the file comment to describe only the entry observation and the handoff marker.
- `internal/loomcli/arm.go`: delete `anomalyDeps`, and the `detectAndFileAnomalies` call at the head of `loomPostRun`; rewrite `loomPostRun`'s and `loomPreStep`'s doc comments so they no longer mention Tier 1 filing.
  `loomPreRun` and `observeStepEntry` keep taking the entry observation; card 4 turns it into a note.
- `internal/loomcli/halt.go`: delete the `detectAndFileAnomalies` call at the head of `loomAfterStep`, the `TierOneFiled` field, its note line and its assignment in `reflectHalt`; rewrite the file comment and `loomAfterStep`'s doc comment so a halt has one path, the halt note and its reflection.
- `internal/loomcli/cli.go`: delete the `fileIssue` field, `refuseFileIssue`, its assignment in `newLoomCLI`, and the `selfreportengine` import, so `loomcli` no longer imports `selfreportengine`.
- `internal/loomengine/anomaly.go`: delete `Anomaly`, `DetectAnomalies`, `countDoneEntries`, `renderHaltTitle` and `renderCrashResumeTitle`.
  `DetectCrashResume` becomes `func DetectCrashResume(entry EntryObservation) bool`, with the same five conditions.
  `detectHaltAnomaly` becomes the exported `func ClassifyHalt(state shedengine.State, reason string) (AnomalyKind, bool)`, keeping its partition: blocked with the `shedengine.ReasonBounceBudgetExhausted` prefix, blocked otherwise, failed, else none.
  Rewrite the file comment and the kind comments: the classifier now feeds friction notes, not issue titles.
- `internal/loomengine/anomalybody.go`: delete; its only caller was the filer.
- `internal/loomengine/config.go`: delete `LoomSelfreportFiled`, `LoomSelfreportFiledLock` and their two filename constants; reword `LoomStepHandoff`'s comment so it no longer cites the filed marker or "a spurious issue".

Tests for removed code are deleted, not adapted:

- `internal/loomengine/anomaly_test.go`: delete the `DetectAnomalies` and title-stability/distinctness/determinism tests; keep `TestDetectCrashResume` against the bool return, and turn the halt rows into a `ClassifyHalt` table covering blocked with the budget prefix, blocked with another reason, failed, and running/paused/awaiting/done as no kind.
- `internal/loomengine/awaiting_test.go`: retarget `TestDetectHaltAnomaly_AwaitingFilesNone` onto `ClassifyHalt`, or fold it into the `ClassifyHalt` table if that covers awaiting.
- `internal/loomengine/anomalybody_test.go`, `internal/loomengine/loomstatus_test.go` (only the deleted accessors) and `internal/loomcli/selfreport_github_test.go`: delete.
- `internal/loomcli/selfreport_test.go`: delete every `detectAndFileAnomalies` and marker test, `filedCall`, `selfreportTestFixture` and `pendHaltTitle`; keep the helpers other files still use (`writeSelfreportStatus`, `productJSON`, `crashResumeEntry`, `haltStatus` if used) and delete any left unused.
- `internal/loomcli/halt_test.go`: drop the fixture's `fileIssue` fake, `filed` slice, `tierOneOn`, `requireFiledOnce`, `requireNothingFiled`, `requireFiledBeforeReflection`, `tierOneLine` and every `TierOne` test; keep the halt-note and reflection tests, and drop producer vars left unused.
  Keep the `handoffSteps` helper: `TestStep_TierOne_HandoffMarkerSuppressesCrashResume` asserts only on filed titles and goes with the filer, and card 4 re-asserts the same two-step voucher behavior on the friction directory over the kept helper.
  `newHaltFixture`'s directory loop calls `LoomSelfreportFiledLock`: drop that entry and reword the comment above the loop, which names the filed-title marker.
- `cmd/lyx/pathrules_test.go`: delete the two `LoomSelfreportFiled` rows.

Run `grep -rn "Tier 1 anomaly\|selfreport-filed\|RenderAnomalyBody\|DetectAnomalies\|fileIssue" --include=*.go .` after the edit; only `contracts/stencils/claims_test.go`'s stencil claim may still match, and card 6 rewrites it with the stencil.

**ImpactSummary:** loom's run and step hooks stop calling GitHub; `loomengine`'s anomaly API shrinks to `DetectCrashResume` (bool) and `ClassifyHalt`, both with no production caller until card 4.

**Commit:** 1: loom: remove Tier 1 GitHub issue filing
