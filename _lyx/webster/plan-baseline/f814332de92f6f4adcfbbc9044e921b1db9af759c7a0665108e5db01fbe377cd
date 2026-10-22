# Card 9 — A request held by a hub pause waits for the resume

**Edit:**
- `internal/orchengine#Watcher.tickIdle`
- `internal/orchengine#Watcher.tickHandoff`
- `internal/orchengine/doc.go`

**Create:**
- `internal/orchengine#Watcher.unpausedSince`
- `internal/orchengine#TestCycleRequest_HubPauseTimeIsNotAge`

**Uses:**
- `internal/orchengine#Watcher.pauseTick`
- `internal/orchengine#newWatchEnv`
- `internal/orchengine#watchEnv.pauseHub`
- `internal/orchengine#watchEnv.resumeHub`
- `internal/orchengine#watchEnv.tick`
- `internal/orchengine#TestCycleRequest_StaleMarkerRemovedAtFirstTick`
- `internal/orchengine#TestCycleRequest_AbandonedWithinTimeoutOfRequest`

**Intent:**
D5's third and fourth bullets: under a hub pause the watcher's `pauseTick` holds the cycle machine, so a request made under a pause longer than `handoff_timeout_s` was removed as stale, or timed out on the first handoff tick, as soon as the pause ended; the orch then deleted `.lyx/orch/cycle-request` by hand.

`Watcher.unpausedSince` reads the watcher's pause record with `pauserecord.Read(w.pause.RecordPath)` and returns `pauserecord.AwakeSince(from, now, coverage.Intervals)` with the `Record.CoverageOf("", now)` intervals (empty slug: hub entries only).
A watcher told no pause record (`w.pause` nil), or a record that cannot be read, discounts nothing, matching `pauseTick`, which reads an unreadable record as uncovered.
The record keeps a resumed entry for `pauserecord.Retention`, so a pause resumed longer ago than that no longer discounts; that is the stated bound, not new code.
`tickIdle`'s stale-request removal compares `unpausedSince(req.RequestedAt, now)` with the handoff timeout instead of `now.Sub(req.RequestedAt)`, and logs that age.
`tickHandoff` keeps its clock start (the request time when it predates the phase entry, else the phase entry) and compares `unpausedSince(started, now)` with the handoff timeout.
Both still remove or time out a request whose unpaused age reaches the timeout, so a stale marker a dead watcher left is still removed.

`TestCycleRequest_HubPauseTimeIsNotAge` is new: a request is written, the hub is paused (`pauseHub`) and the clock moves past twice the handoff timeout while ticks are held, the hub is resumed (`resumeHub`), and the next tick starts the requested cycle with the marker consumed; further ticks inside the timeout of unpaused time keep the phase in handoff-requested, and the cycle reaches clearing once the note gate passes, driven through the test file's existing note helper.
Each discounted clock has its own row, so neither discount can revert unseen:
a row resumes the hub with no turn end read, so the request stays idle, then moves the clock one handoff timeout of unpaused time past the resume and finds the marker removed as stale by `tickIdle`;
a row pauses the hub after the requested cycle entered handoff-requested, holds it past twice the timeout, resumes it, and finds the phase still handoff-requested on the next tick, then timed out to idle with `handoff timed out` once the clock passes the timeout of unpaused time, so `tickHandoff`'s discount is the pause alone.
A row with the pause record replaced by unreadable bytes discounts nothing: a request older than the timeout is removed as stale at the first tick.
`internal/orchengine/doc.go`'s handoff-requested timeout sentence and its stale-request rule gain the excluded hub pause time and the `Retention` bound.
`TestCycleRequest_StaleMarkerRemovedAtFirstTick` and `TestCycleRequest_AbandonedWithinTimeoutOfRequest` keep passing unchanged with no pause recorded.

**ImpactSummary:** Two timeout comparisons inside the watcher read the hub pause coverage it already reads each tick; nothing outside orchengine changes.

**Commit:** 9: a cycle request held by a hub pause waits for the resume
