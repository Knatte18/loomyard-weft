# Card 3 — Resize re-apply failure logs once per episode

**Edit:**
- `internal/reedengine#watchState`
- `internal/reedengine#Engine.handleWatchOutcome`
- `internal/reedengine#TestWatchLoop_NonSentinelFailureDoesNotGoDormant`
- `internal/reedengine#Engine`
- `internal/reedengine#Engine.repairSessionName`
- `internal/reedengine#TestRepairNames_SessionName`
- `internal/reedengine/doc.go`

**Intent:**
D3: the watch loop logs `reed: resize re-apply failed` at Warn on a session's first failure after a success or after its watcher starts, at Debug while the failures go on, and one Info line when the re-apply recovers, with the count of failed ticks.

`watchState` gains a `resizeFailures logger.Episode` field.
A `watchState` is built once per `watchLoop` call, so the episode lives in the session's watcher, in memory only, and ends with it: a session that ends mid-episode owes no recovery line, and a respawned daemon starts with no episodes.
In `handleWatchOutcome`, the non-sentinel error branch calls `state.resizeFailures.Fail("reed: resize re-apply failed", "socket", …, "session", …, "err", err)` in place of its `logger.Warn`;
the real, non-deferred success path (after the `res.Deferred` return) calls `state.resizeFailures.Recover("reed: resize re-apply recovered", "socket", …, "session", …)`.
A deferral is neither a failure nor a recovery, the `errWorktreeRootGone` sentinel keeps its own dormancy logging, and the `abandoning this resize event` Warn stays, since it fires once per resize event.
The episode spans both poll and signal mode, since `state` is passed in either.
`doc.go`'s account of the watch loop's failure handling names the episode rule.

`TestWatchLoop_NonSentinelFailureDoesNotGoDormant` keeps its poll-mode setup and its dormancy assertion, captures at Debug (`logcapture.Capture` plus `logger.SetVerbosity(2)`, process-global, so the test stays serial), and adds the episode sequence:
with `select-layout` failing on every tick, once at least five `list-panes` calls ran, the log holds exactly one `level=WARN` line and at least one `level=DEBUG` line carrying `reed: resize re-apply failed`;
the fake then answers `select-layout` with success, and the test waits for exactly one `level=INFO` `reed: resize re-apply recovered` line carrying `failed_ticks=`;
the fake then answers `display-message` with a new size (`150 55`) and `select-layout` with an error again, since a recovered watcher records the applied box and re-applies only on a changed one, and the test waits for a second `level=WARN` failure line.
The doc comment names the episode sequence it now pins.

D11, the watchdog's other per-tick Warn in this package: `repairSessionName`, reached on every watchdog tick while a session's name has drifted and its pane is idle, logs `reed: operator activity unreadable, not holding the session rename` at Warn on each tick while tmux cannot list its clients.
`Engine` gains an `activityFailures logger.Episode`, and `repairSessionName` calls `e.activityFailures.Fail` with that message and fields on a client-list error and `e.activityFailures.Recover("reed: operator activity readable again for session name repair", …)` on a successful read; the rename decision is unchanged.
`TestRepairNames_SessionName`'s "UnreadableClientListStillRenames" row runs the repair twice on one engine, still wants the rename, and asserts exactly one `level=WARN` `operator activity unreadable` line, capturing the process-global logger at Debug, so that row runs serially.

**ImpactSummary:** one field on each of `watchState` and `Engine` and two method bodies in `internal/reedengine`; the log texts are unchanged, so only their level per tick and new recovery lines change, and no layout or rename decision does.

**Commit:** 3: reed: log a failing resize re-apply once per episode
