```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F2, F5; F3, F4, F6, F7 are test and wording additions).

**1. MEDIUM — F5, `internal/dobbyengine/view.go` `slotRuns`: the view and the scheduler can disagree when someone else holds the driver lock**
- The proposal counts a done run in the slot count whenever `DriverLockHeld` is true.
- The scheduler keeps a done run only when its own pass is in `d.flights` (`passes.go:150-153`: `run.State == runStateDone && !inFlight` is deleted).
- The driver lock is also held by a hand-run `lyx batten run <slug>` on a done run, and by a probe collision (see 2).
- In that case `d.flights` is empty. The scheduler drops the run and starts the next task, while `BuildView` shows that task `AtMaxRuns`.
- That is the mismatch the review's own F5 premise forbids: the view explaining a wait the scheduler does not have.
- The new row "ADoneRunHoldingItsDriverLockStillTakesASlot" pins the one-sided case only.
- `DriverLockHeld` cannot tell the dobby's pass from anyone else's, so the proposal can only narrow the gap, not close it. At minimum the comment in `view.go` should say the count is an approximation.

**2. LOW — F2, `lock.ProbeGrace` fix: two probes can still collide**
- The grace is applied only to `DriveVerb` and `dobbyengine.Run`.
- The probes themselves (`lockHeld`, `Live`) still use `TryAcquireWriteLock`.
- A pane probe that overlaps `ensureOne`'s `Live` probe (`lyx orch start`, or a dobby tick) makes the second one read "live" for a dead dobby, so `ensureOne` skips the spawn (`start.go:157-158`).
- The grace does not cover this path.
- A pane probe that overlaps a driver-lock probe also sets a false `DriverLockHeld`, which feeds the F5 slot miscount in 1.
- Both are transient, but they show the pane's probes still disturb other probes, which the grace does not fix.

**3. LOW — F1, `worktreeHasDobby(..., hubgeom.DobbyStart(location).Beams)`: an unrelated pair's failure hides a valid beam's section**
- `localBeams` (`hubgeom/dobbygeom.go:78-108`) fails the whole list on any pair's unreadable origin record (`ReadOriginFor` error) or owner resolution error (`owners.Effective`).
- A legitimate beam's pane then draws no dobby section, with only a Warn logged, because another pair is broken.
- It also runs `ReadOwners` and a `ParentOf` lookup per pair at pane start, work the section does not need.
- A narrower check would read only the acting worktree's origin record (`fabricengine.ReadOriginFor(prime, acting)` and `origin.Beam`).

Tool calls: none. My context already held every file I needed.
