---
round: 1
focus:
  - "`tools/sandbox/SANDBOX-CREEL-SUITE.md`: both sides edited it, the task in the pre-session checklist and the parent in the scenario `Watch` lines. Does the merged file read consistently, and does the task's removal of the `command -v lyx` check leave any parent-added scenario text still relying on it?"
  - "`cmd/lyx/testdata/test-budget.yaml`: the parent raised `internal/shuttleengine` and the task raised `internal/reedcli`. Does each package's merged top-level test count fit its row, and does any budget row for a package both sides touched (`internal/shuttleengine`, `internal/shuttleengine/claudeengine`) need an edit the merge did not make?"
  - "`pattern/PATTERN-daemon-wakeups.md`: the parent changed the unbounded-wait list. Does the task's reed watchdog (`internal/reedcli/watchdog.go`, `internal/reedengine/watchdog.go`) follow the rule, including its stop-request exit and bounded wait in `reed down`, and is the watchdog listed or exempt as the pattern requires?"
  - "The parent's new client behavior, a connect wait after spawn that ends on the exit of the daemon it started (`internal/creelengine/client.go`), against the task's changes to `internal/shuttleengine/claudeengine/command.go` and the reed pane-shell path: does anything the task changed about how a pane's shell or `lyx` binary is resolved break creel's spawn or its `LYX_BIN` assumptions?"
  - "`internal/shuttleengine` (parent: `run.go`, `session.go`, `engine.go`) against `internal/shuttleengine/claudeengine` (task: `command.go`, `claudeengine.go`): do the parent's session and run changes still match the task's changed command and env-file contract?"
  - "Test economy: do the parent's new `internal/shuttleengine/startup_test.go` and the task's new `claudeengine/envfile_integration_test.go` cases cover the same behavior, and is any interaction of the two sides' changes left without a test?"
---

Seeded from the overlap list and the two diffs.
