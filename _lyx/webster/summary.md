# Orch `start --continue`: session id exposed, recorded and resumed

Batch `01-shuttle-session-id` ran as one fork and recorded `done` at head `4158bfa40a1da6593e982da40723b5043dde412c`.
The fork committed the plan's four cards in order: the shuttle session id, the watcher recording it, `start --continue`, and the `status` report.

The fork reported one deviation from the plan.
In card 1, the existing `TestReadSessionStates` rows with no record id assert `recorded-session` rather than an empty `SessionID`, because the `writeReadRun` helper defaults an empty record id to that value.

The scope guard's digest listed three files changed outside the batch's own targets: `internal/orchcli/start.go`, `internal/orchcli/start_test.go` and `internal/orchengine/watcher.go`.
`record-batch` raised no warnings, and the deviations are informational.
