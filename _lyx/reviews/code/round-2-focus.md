---
round: 2
focus:
  - "internal/dobbyengine/passes.go planPass and triggerFor: when a run was covered by a pause and the pause lifts, triggerFor sets Lift with covered=false; settle whether the pass planned is a full pass (passFullNoReopen when run.OriginOwed) in the uncovered tail, and whether the passes_test.go lift rows plan with the inputs triggerFor really produces."
  - "internal/dobbyengine/passes.go liftPending lifecycle: settle whether a lift seen while the run is not running, carries a cancel marker or is no longer listed leaves a stale liftPending entry, and whether an in-flight pass keeps it."
  - "internal/boardengine/board.go boardCriticalSection and internal/boardengine/draft.go migrateDraftStatus: settle whether a release (null status) that is the first write to a board without draft_status stays released, and whether applyDraftRule still runs after the mutation."
---
