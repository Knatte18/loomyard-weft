```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. Traced every path that already writes or reads the state the record changes.

**Paths traced, per decision:**

- **D1 `landed` (batten record `done`):** dobby's `clearDoneRuns` → `battencli.ClearDoneRun` removes a `done` batten record and its task branch. The `landed` state survives only until that clear runs. After it, the record defines no state for the slug. **Breaks**; see finding 2.
- **D1/D3 cancel-marker readers:** `dobbyengine/passes.go:124,203`, `runnotice.go:182`, `waitreminder.go:72` and `ended.go:37` all read `run.CancelMarker`. D1's rule sends every reader to `internal/runfsm`, and D11 names `ended.go`. **Survives.**
- **D3 cancelled record kept:**
  - `battencli` cleanup's `judgeStatus` removes only `done` runs. It refuses other states with "lym batten cancel ends it, and a later cleanup removes it".
  - `ClearDoneRun` removes any `done` run and deletes its task branch on origin.
  - D3 never says which shed state a cancelled record carries. **Breaks either way**; see finding 2.
- **D3 origin-owed marker retired:** the continue-pass refusal in PATTERN-owner-machine reads it. D6's catch-up replaces that refusal. **Survives.**
- **D6 step 4, removing the local pair of a run owned elsewhere:** the fabric reconcile never hands an owned_elsewhere pair to teardown (`fabricengine/doc.go:162`). The precedent the record cites does not exist. **Breaks**; see finding 3.
- **D6 sleep detection:** `pauserecord.SleepWatch` is used in `dobbyengine/loop.go`, `hibernateengine/watch.go`, `shedverbs/loopwatchdog.go` and `shuttleengine/run.go`, but not in the orch watcher. D6 adds it there. **Survives**, as new work.
- **D7 pause callers:**
  - `pauserecord/record.go:23-32` defines `OwnerCaller`, `AdoptCaller` and `HoldCaller = "adopt-hold"`.
  - `dobbyengine/passes.go:127` withholds a pass when `run.Held` is set.
  - `battenshed/innerrun_parentlock.go:36` and `pairteardown/followup.go:58` read `OwnerHeld`.
  - The retired callers lose their only triggers, and the act rule replaces them. **Survives.** The name collision with the new `hold` caller is finding 1.
- **D7 parent-lock exemption:** `pauserecord.WaitUncovered` exempts a caller holding a parent lock from slug entries, as D7's bound says. **Survives.**
- **D2/D11 tender pushes:** in `tendercli/deps.go`, `PushPair` (`SpawnPairPush`) retires per D11 and `PushPrimeRecords` stays detached per D2. **Survives.**
- **D2, the teardown-lock respawn:** `fabricengine/teardownlock.go` `respawnPairPush` matches D2's description. **Survives.**
- **D4/D5 against the reconcile:** `landingshed/finalize_owner.go` `ownerChecks` judges `owned_elsewhere` from `ReconcileOrigin`. D5 routes that comparison through the registry, and the Finalize carve-out stays. **Survives.**

**Findings:**

1. **LOW · consistency · D7, the third bullet** ("one new caller `hold`").
   `pauserecord.HoldCaller` already exists, with the value `"adopt-hold"`, the legacy caller D7 retires. A new caller named `hold` beside a retired constant named `HoldCaller` invites mixing the two up in the migration D7 describes, which reads `adopt-hold` entries.
   Evidence: `internal/pauserecord/record.go:32`, `internal/pauserecord/doc.go:21-24`.
   Fix: name the new caller's constant apart from `HoldCaller`, or rename the retired one.

2. **MEDIUM · design · D3, cancel intent bullet, and D1's `landed` state.** D3 says the batten record "goes with the pair's teardown" and is kept otherwise. It is silent on the two paths that already remove batten records:
   - The dobby's done-run clear (`dobbyengine/donerun.go` → `battencli.ClearDoneRun`) removes any `done` batten record. It also archives and deletes the task branch locally and on origin.
   - `lym batten cleanup` removes `done` records and refuses every other state, naming `lym batten cancel` as the way forward.

   D3 never states the shed state a cancelled batten record carries:
   - If the finish sets `done`, the dobby clears a kept cancel's record and deletes its task branch. The operator's keep is lost, and D11 reads the kept pair as an orphan.
   - If it stays non-`done`, cleanup refuses it forever and names a cancel that has already finished.

   D1's `landed` likewise exists only until the clear runs; after it, the slug reads no state.
   Evidence: `internal/battencli/cleanup.go:126-149` (`ClearDoneRun` doc: removes the record and the task branch whatever the recipe), `:542-551` (`judgeStatus`), `internal/dobbyengine/loop.go:92-96`, `internal/beamcli/dobbydeps.go:177`.
   Fix: state the cancelled record's shed state, which remover may clear it, and what D1 reads once a landed record is cleared.

3. **MEDIUM · design · D6 step 4** ("removes the local pair only, as fabric's removal under an owned-elsewhere verdict does").
   No such removal exists. The reconcile never hands an owned_elsewhere pair to the teardown seam, and the move for such an entry is fast-forward only. A pair's teardown must go through `internal/pairteardown` (PATTERN-pair-remote). That composite settles the board claim unless `LeaveClaim` is passed, and its archive step pushes and deletes the weft branch. So the "local only, origin's branches and the board claim stay" removal D6 needs has no existing route, and the record names a false precedent.
   Evidence: `internal/fabricengine/doc.go:162`, PATTERN-pair-remote background "Pair teardown" (claim settling; archive and delete of the weft branch).
   Fix: name the removal as a `pairteardown` request that leaves the claim and deletes no origin branch, a mode the design adds.

**Tool calls:**

1. Read `dobbyengine/ended.go`, `battencli/cancelcmd.go` `RemoveRecords`, and grep for `removeRunRecords`: to find every remover of batten records, which my context lacked.
2. Read `battencli/cleanup.go` at the done-run removal and the cleanup candidate: to see which states those removers take.
3. Read `judgeRunDirectory`, `judgeStatus` and `clearDoneRun`: for their state conditions.
4. Grep for `clearDoneRun` callers and cancel-marker readers: to find the dobby path and the readers of the retired marker.
5. Read the `ClearMotifRun` and `ClearDoneRun` docs and their dobby callers: to confirm the dobby clears done records and task branches.
6. Grep for the pause caller constants and `Coverage` readers: to check the D7 caller names and their readers.
