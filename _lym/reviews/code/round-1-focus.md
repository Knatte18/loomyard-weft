---
round: 1
focus:
  - "Scope against the eight named tests: the commit parallelizes six; check whether `TestWorktreeSeam_Events` and `TestReconcile_MigratesMarkerlessWefts` are left serial with their cause named in `_lym/landing/summary.md`, as the task requires for a cause that cannot be removed."
  - "`failedpush.go` and `installedPushNotifier`: the notifier falls back to the installed package-level one. Settle whether any path a parallel test reaches still reads that variable, and whether every other test that sets it moved to the new `SyncOptions` field."
  - "`anchormove.go`, `topology.go`, `housekeeping.go`, `originreconcile.go`: settle whether production behavior is unchanged and only the source of each input moved, including zero-value defaults of the new `Topology` and `ReconcileOptions` fields and nil-hook handling."
  - "`export_test.go` and `housekeeping_integration_test.go`: settle whether removing the package-level setters and moving the log-reading rows to `TestStoreHousekeeping_Warnings` drops or loosens any assertion."
  - "Remaining serial tests: settle whether any test outside the eight, still serial or now parallel, sets or reads state (environment, working directory, shared path, remaining package variables such as `originCallHook` and `gcAutoThreshold`) that a newly parallel test can see."
  - "Callers of changed signatures (`addanchor.go`, `remove.go`, `pushclassified.go`, `fabric.go`): settle whether every caller is updated and nothing outside `internal/fabricengine` changed beyond that."
---

Seed focus for round 1, from the decision record's task and limits and the commit message of the review range.
