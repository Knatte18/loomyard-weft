# Fixer report, round 1 (code)

## Changes

- **F1 (MEDIUM), fixed as proposed**, commit `ac5ca284`.
  `internal/beamcli/dobbydeps_integration_test.go` gains `TestHubDobbyDeps_WaitReminderPostReachesTheSlackRelaySignedAsTheDobby` and a `startCreelDaemon` helper.
  The test runs creel's daemon in process over the hubforge hub, calls `WaitReminders.Post` of the prime's dobby and of a beam's dobby, and compares the daemon's `posted` log entries exactly: sender `<shortname>:DOBBY` or `<shortname>:task1:DOBBY`, recipient `<shortname>:SLACK`, and the body.
  The review pointed at reading the relay's mailbox; the test reads the daemon's log instead, because creel refuses a post to a receiver that is no known strand, so the fake delivery holds the relay as a live strand, whose mail the log shows and a read would not.
- **F2 (LOW), fixed as proposed**, commit `9c23a5a8`.
  `internal/dobbyengine/waitreminder_test.go` gains two rows in `TestWaitReminders`: `ARunWithACancelMarkerIsSkippedAndRecordsNoWait` (no post, empty `OperatorWait`) and `AStandingAnchorChangeStopsTheWaitCheck` (a record in `geom.MoveRecords`, no wait check read).

Gate: `lym gate test` tier 0 over `./internal/dobbyengine ./internal/beamcli` and tier 1 over `./internal/beamcli` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
