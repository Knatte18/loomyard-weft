# Lens review

Every lens returned a report.

- forced-edits, BLOCKING, card 9: `RemoveWarpWorktreeDirForTest` (`internal/fabricengine/export_test.go:68`) aliases the re-signed `removeWarpWorktreeDir`, and `destructivegaps_integration_test.go:204,267` call it with four arguments — fixed: the alias becomes a wrapper passing a zero `leftoverSeam`, so its callers compile unchanged.
- forced-edits, LOW, card 6: `hookStateSkillEngine`'s comment and the hook-state row name describe a typed first load — fixed: the member joins the card and the row is renamed.
- forced-edits + row-reachability + criteria-coverage, LOW, card 8: the readiness table hard-codes `CaptureQueue: []string{"gate"}`, so "blank captures" was not scriptable — fixed: a `captures` field, blank captures on the new ready row and the reused died row, and the doc comment updated.
- forced-edits, LOW, cards 4 and 7: both edit `docs/overview.md` — fixed: card 7 says it edits the skills bullet, a different line from card 4's.
- criteria-coverage, MEDIUM, card 6: no row colors a strand with no skills — fixed: row `NoSkillsColorsRightAfterStartup`.
- row-reachability, MEDIUM, card 6: no row fails if the color is typed before the launch-carried turn ends — fixed: `skillReed` records `ColorAfterLaunchEnd` per color send, and every launch-carried colored row asserts it, the `DelayLaunchEnd` row included.
- criteria-coverage + row-reachability, MEDIUM, card 3: no row reaches `ResizeSignal`'s SIGWINCH forwarding — fixed: a non-parallel subtest signals its own process, asserts one value, then asserts none after `stop`; on Windows it asserts the nil channel.
- criteria-coverage, MEDIUM, card 4: no row takes a relative path, so a join against the wrong base passes — fixed: a row through `clihelp.ExecuteIn` (`internal/clihelp/exec.go:220`) with a subdirectory cwd and a bare file name.
- row-reachability, LOW, card 7: the over-limit row with an empty `Config` hits the TTL refusal first (`prepare_test.go:71`) — fixed: the row carries `cfg: templateConfig(t)`.
- test-economy, LOW, card 9: the held-open-file row repeats the success row off Windows — fixed: the row runs on Windows only.
- test-economy + criteria-coverage, LOW, card 9: the integration permission row repeated the unit retry decision, and the composed `Remove` error went unpinned — fixed: the row asserts the composed message and `errors.Is(err, os.ErrPermission)`, with the removal error wrapped by `%w`.
- test-economy, LOW, card 9: the recording assertions could be a subtest of `TestGate_RecordOnlyOnObservedEffect` — rejected: the retry behavior needs its own test, and asserting its own executor's record there is one assertion, not a repeated test.
- criteria-coverage, LOW, card 9: the target-no-longer-on-disk branch has no row — rejected: that branch keeps today's message unchanged.
- test-economy, LOW, card 8: the trust-gate row repeats the existing dismissal row — rejected: its point is that a session start with no turn start is not ready under the new events-first answer, which the existing row (ready afterwards) does not show.
- test-economy, LOW, card 6: `ColorFollowsATimedOutLoadTurn` repeats the timeout path — rejected: it is a row, and the color following a timed-out turn is a D2 criterion the timeout row does not assert.
- row-reachability, LOW, card 3: `NilSleepWakesOnAResize`'s drain relies on `Run` sleeping after the cancelling poll — rejected: card 3 keeps `Run`'s poll-then-sleep loop shape (`internal/statuspane/pane.go:69-99`).
- row-reachability, LOW, card 4: no row runs `fmt` through the pre-run — rejected: `TestResolveRunID` is the Tier-1 seam for the pre-run's run-id step, as `arm.go`'s `resolveRunID` comment states, and its new row pins the fix.
- criteria-coverage, LOW, card 1: the doc-comment criterion has no test — rejected: it is a comment, checked by review.
