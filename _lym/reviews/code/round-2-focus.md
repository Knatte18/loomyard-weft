---
round: 2
focus:
  - "Test economy: the redundancy report lists `TestOwnerPreflight_TeardownResume` (internal/battencli/owner_test.go) as contained, covered by `TestRunCmd_ResumeDispositions`, `TestOwnerPreflight_GoneTaskWorktree` and `TestBattenIntegration_Rows`. Compare its assertions with theirs and settle whether it pins anything they miss."
  - "Test economy: `TestRun_ALaterArchiveFailureLogsAgainAfterTheOwnerLeaves` (internal/dobbyengine/handover_test.go) is listed as contained, covered by `TestRun_ForeignOwner` and `TestRun_LockOwnerAndWakeups`. Compare assertions and settle whether it pins a log or ordering the covering tests never check."
  - "Test economy: `TestPlanPass` (internal/dobbyengine/passes_test.go) is listed as contained, covered by `TestDrivePasses_ExitHandling` and `TestStartReady_RefusalsAndBackoff`; `TestReadyToStart` (internal/dobbyengine/starts_test.go) is listed as contained, covered by `TestStartReady_RefusalsAndBackoff`. Compare assertions for each and settle whether to fold, delete or mark `//testtiming:keep`."
  - "Machine-name comparison: every comparison of machine names in the diff must go through the registry. Check `Registry.sameMachine` and its callers in internal/boardengine, internal/tenderengine and internal/dobbyengine for a plain name equality that misreads a renamed machine's former name as another machine."
---

Round 1 left the contained-test comparison undone; the first three entries ask for it.
