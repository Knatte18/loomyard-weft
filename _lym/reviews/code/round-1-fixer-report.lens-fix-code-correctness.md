```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fixer's changed code (F2–F8) through the fix-code-correctness lens.

**Findings**

1. **LOW**: `passes_test.go`, row `PausedReachedParentHeldGivesNone`. The row passes with the defect in place, so it pins nothing.
   - It uses `probe` with `covered=true`. With the `run.Held` check removed from `planPass`, a paused reached-parent run still gets `passNone`. The covered branch returns none because `run.State != runStateRunning` and the trigger is not a lift.
   - It also gets `passNone` if `drivable` wrongly excludes paused runs.
   - The existing running-run row `HeldLiftTriggerStillCoveredGivesNone` uses `passTrigger{Lift: true}`. Without the `Held` check, a paused reached-parent run under a lift would get `passFull`.
   - Fix: change the trigger to `passTrigger{Lift: true}`.

2. **LOW**: `dobbypairanchor_test.go`, `TestDobbyRuns_ReadsWhetherAPausedRunsWorkReachedItsParent`. F2's third case is untested: an unreadable judgment gives `ReachedParent` false with no error from `DobbyRuns`.
   - The fixer dropped the `unreadablekid` fixture because the pair's parent holds the work (the gate read `true`).
   - Deleting the Warn-and-leave branch, which would return the error from the closure, fails no test.
   - The other cases are sound:
     - `teardown` fails if the `found` argument is wrong.
     - `running` fails if the paused gate is dropped.
     - `finalizedkid` exercises the child-history path.
   - This is a known gap. The fixer should name it in the report as not covered, not as fixed.

3. **LOW**: `dobbyreads.go`, `unreadableReachedParent` (F4). The episode is closed only by a successful read in the paused branch.
   - If a run leaves paused while its judgment is failing, the episode stays open.
   - A later failure for the same slug then logs at Debug instead of Warn.
   - It is harmless and bounded, since the key is the slug.

**Traced and correct**
- The F5 subtest runs as follows:
  - Tick 2: the lift is set, `DriverLockHeld` gives none, and `drivable` is true, so the pending lift is kept.
  - Tick 3: `Lift` gives `passFull`, a full pass.
  - Under the old `State != running` cleanup, tick 2 deletes the pending lift and tick 3 spawns nothing. The test therefore fails with the defect in place.
- The `OwnedElsewhere` row returns `passFull` on an uncovered probe tick, and it fails if `drivable` excludes paused runs.
- Format arguments:
  - `claim.go`: three `%s` with three arguments.
  - `anchor.go`: two `%s` with two arguments.
- The `EpisodeSet` zero value is usable as a package variable. `Recover` on a key with no episode does nothing, so the success path logs nothing.

**Tool calls:** none. The context already held every file.
