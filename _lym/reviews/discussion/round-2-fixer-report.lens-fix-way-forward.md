```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward: no BLOCKING findings, three MEDIUM and one LOW.

Ways forward checked:
- D7 step 1: a failure to end the process clears the mark.
- D7 step 4: a stop during the window means the relaunch removes the strand and writes nothing.
- D7: a check refused for no transcript reruns steps 2 to 4 with a fresh launch.
- D7: the mark's bound ends the run as a strand that died once twice the startup timeout has passed.
- D7 orch paragraph: restart and fresh start through the `StartRelaunch` case.
- D6: the start path carries the copy fact into the new run directory.
- The new D6 and D7 acceptance rows.

1. **MEDIUM. The fresh-launch fallback keeps the old session id on the reused GUID.**
   - Changed text: D7, "A check refused for no transcript runs steps 2 to 4 again with a fresh launch: a new session id, the run's skills and the run's prompt."
   - Step 4 now has reed replace only the stored `Cmd` and `ResumeCmd` under the same GUID. Neither reed's stored session id nor `RunState.SessionID` is said to change.
   - Before this fix, a new strand made through `AddStrand` carried `SessionID: launch.SessionID` (`internal/shuttleengine/run.go:678`), and the run record set `SessionID: launch.SessionID` (`run.go:709`).
   - On the fresh-launch path, the run record and reed would still name the abandoned session. The session-end usage record, the liveness read and the transcript readers would then follow the wrong session.
   - D6's refused-resume fallback goes through this same fresh relaunch, so it inherits the gap.
   - Way forward: step 4 should also replace the strand's session id in reed and write `RunState.SessionID` when the launch's session id differs.

2. **MEDIUM. The orch restart cannot reach the `StartRelaunch` case from the state it names.**
   - Changed text: D7's orch paragraph, "both taking the orch start path's relaunch branch (the `StartRelaunch` case in `internal/orchcli/start.go`)".
   - On a switch the limited orch strand is still live and the watcher calling the capability is itself live. `DecideStart(strandLive, watcherLive)` then returns `StartAlreadyRunning`, not `StartRelaunch` (`internal/orchengine/launch.go:30-38`).
   - The start path also first calls `orchengine.WaitWatcherGone` (`start.go:681`), which would wait out its timeout on the calling watcher.
   - The text does not say the capability runs the case's body directly, outside `DecideStart` and the watcher-gone wait. As written, the restart either does nothing or stalls.

3. **MEDIUM. Step 1's failure path can overwrite a concurrent stop.**
   - Changed text: D7 step 1, "A failure to end the process clears the mark back to running over the live strand."
   - Step 4 guards its own record write by re-reading the record in one hold of the lock and yielding to a terminal outcome. Step 1's clear back to running has no such guard.
   - A `StopStrand` that writes stopped between the mark and the failed end would be overwritten with running. That is the same overwrite F20 raised for step 4, now moved to step 1's failure path.
   - Way forward: step 1's clear should take the same lock-held re-read.

4. **LOW. A stop after the driving `Wait` has died can leave a held dead strand in reed indefinitely.**
   - Changed text: D7 step 1, "held from reed's reconcile and resume until step 4", together with the new criterion "a `StopStrand` during a relaunch leaves the run stopped and no strand up".
   - `stopRecorded` removes a strand only when `strandRemovable` holds: live, or own-window and present (`internal/shuttleengine/stop.go:135-137`). A held window-0 strand with a dead pane is skipped.
   - If the driving `Wait` has died, nothing reaches step 4. The record now reads stopped rather than relaunching, so the bound path never fires either.
   - The strand then stays held in reed's table indefinitely, and the orphan sweep keeps its directory because the GUID is still tracked.
   - Way forward: the stop verb should remove a strand held for relaunch whatever its pane state.

Tool calls beyond context:
- Read `DecideStart` in `internal/orchengine/launch.go`, to check which start case the watcher's capability reaches. Neither the orch relaunch decision nor `WaitWatcherGone`'s behaviour was in context.
