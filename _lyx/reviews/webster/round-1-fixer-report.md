# Webster round 1 — fixer report

All six findings are fixed, one commit each, none pushed.

- **F1** (`afc510c9c`) — `removeStatusStrands` now logs in the logger's key/value style: `"guid", s.GUID, "cause", err` on a failed removal and `"cause", err` on a failed status read, with `loomcli:`-prefixed messages matching the package's other warnings.
- **F2** (`a2ccd44ea`) — the `first remove fails` row uses the guids `guid-a`/`guid-b`, asserts the log contains `guid=guid-a` and does not name `guid-b`, and the `status fails` row now also asserts `cause=boom`.
  The previous printf-style code fails this test.
- **F3** (`c7b8cc23d`) — every new or changed comment the review listed now uses semantic line breaks.
  Only the touched sentences were re-broken; untouched legacy lines stay as they were.
- **F4** (`97e03d7ed`) — the early-probe comment reads "seed, commit and bring up reed".
- **F5** (`212a78e8c`) — the start_watchdog_test.go header says the call sits after `c.reed.Up()` and the strand branch, which run `Up` and the branch's `Status` on a concrete engine.
- **F6** (`5d9a9f7e6`) — `stepSmokeTimeout` is renamed to `verbSmokeTimeout`, and its doc now covers both step and `start --no-attach`.

Verification after the last fix: `go build ./...`, the plan's unit and integration test commands, and `go vet` under the default, `smoke` and `integration` tags all pass.
The smoke tests in smoke_drivingsurface_test.go also pass against a live tmux server, run with `$TMUX` unset.

## Deferred

None.
