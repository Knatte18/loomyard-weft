```yaml
lens: fix-proposal
outcome: reported
```

Findings (fix-proposal lens):

1. **MEDIUM** — F1, `refuse` in `internal/pawlengine/beamawait.go:268` now returns `awaitWake{acted: true}` when the `SetStatus(NameBeamAwait)` write fails.
   - **Summary:** A persistent status-write failure turns the wait into a one-second loop that re-runs the full landing re-check on every wake.
   - **Evidence:** The failed write keeps the request. Each next wake goes `wake` → `accept` → `SetStatus(Parent-Align)` → `landChecks`, which writes the stop marker, runs `StopDobby` and `Recheck`, and may run `WriteSnapshot`. `acted: true` makes `Call` use `nextAwaitWait(wait, true) = awaitFloor`, so the doubling toward a minute is gone for as long as the status writes keep failing.
   - **Cost of the proposal:** `Recheck` can be expensive, since it checks the beam's members. The test row pins `seconds(1, 1)` and so asserts the floor, but no row covers a failure that lasts more than one wake.
   - **Alternative:** Keep the backoff and skip the re-check while the hold is stuck. Or retry only the lift at the floor, without re-running `landChecks`.

2. **LOW** — F2, `noticeDownDobby` and `refuse` in `internal/pawlengine/beamawait.go`: `refuse` clears the recorded notice even when `StartDobby` failed.
   - **Summary:** A failed restart leaves the dobby down, but the cleared record lets the next re-entry post a second dobby-down notice.
   - **Evidence:** In `refuse`, `clearDownNotice` runs unconditionally after the request removal, before the `restartErr` branch. When `restartErr != nil` the dobby is still down, and the refuse notice already says to run `lyx beam start`. The next driver re-entry finds the dobby down, finds no record, and posts again.
   - **Alternative:** Clear only when the restart succeeded or the dobby was seen live.

3. **LOW** — F2, `noticeDownDobby`: the recorded notice is cleared only when the dobby is observed live at a driver re-entry.
   - **Summary:** If the dobby is restarted and goes down again between two re-entries, the second episode posts nothing.
   - **Evidence:** The `live` branch is the only place the record is cleared, besides `refuse`. The proposal's "cleared when the dobby is seen live" depends on re-entry timing, so the one-notice-per-episode claim in the doc comment holds only per observed episode.

4. **LOW** — F2, `recordDownNotice` in `internal/pawlengine/landrequest.go`: it truncates the await marker with `writeMarker` and then appends the notice line.
   - **Summary:** The record is written in two steps, not atomically.
   - **Evidence:** A crash between the truncate and the append leaves the marker without the line, which only causes a repeat notice. `clearDownNotice` also rewrites the marker with `writeMarker`, so concurrent access is not a concern, since only the driver touches it. The impact is minor.

5. **LOW** — F3, `internal/orchcli/startdobbies_test.go:22`: the comment still says "so the orch start is a resume".
   - **Summary:** One stale "orch start" remains in a comment the fix did not touch.
   - **Evidence:** The grep over `internal/orchcli` and `internal/orchengine` found only this test comment as a stale use. The review named three comments and all three were fixed. The other grep hits describe starting the orch session itself, which is not the removed command, so they are not stale.

Tool calls made (1):
- `grep` for "orch start" wording in `internal/orchcli` and `internal/orchengine`: my context had only the three sites the review named, so it could not show whether others remained.
