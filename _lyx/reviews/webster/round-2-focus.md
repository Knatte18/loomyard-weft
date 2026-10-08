---
round: 2
focus:
  - "Test parallelism: in the tests the diff adds in `internal/shuttleengine/claudeengine/usage_test.go` and `internal/shedadapters/burler_test.go` and `internal/shedadapters/bouncerfacts_test.go`, does each new test either call `t.Parallel` or carry a comment at the test or atop its file naming the process-global state (such as `HOME` via `t.Setenv`, or the global logger via `logcapture.Capture`) that forbids it, per `PATTERN-test-speed`?"
  - "Fan key loading: in `validateFanKeys` in `internal/loomengine/config.go`, does `LoadConfig` still behave as before the diff when `discussion_fan` and `plan_fan` are both empty, including for callers such as `internal/loomcli/arm.go` when `burler.yaml` is malformed or unreadable?"
  - "Dead and self-referential code: in `internal/shuttleengine/claudeengine/usage.go`, does every function the diff adds have a caller, and does each doc comment define its function without leaning on another symbol that is itself unused?"
  - "Formatting: are all Go files the diff touches gofmt-clean, including struct field alignment in `internal/loomengine/config.go`, `internal/loomengine/config_test.go` and `internal/shuttleengine/claudeengine/usage_test.go`?"
---

The first entry restates the `PATTERN-test-speed` rule against the diff's new tests.
The others are carried-over questions on code that changed since the last review, so they ask for a fresh read of the current state.
