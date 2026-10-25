```yaml
lens: bound-breaker
outcome: reported
```

**Claims attacked**

1. D3/D4: "a run another machine takes over is judged by the origin reconcile of a full pass" and "It drives only runs this machine owns" — **broken**.
2. D4: spawn hold "returns `Yield` instead of sleeping" with the owner triggers "persist in the memory record" / "fire across passes" — **broken**.
3. D4: "the flag skips only the origin reconcile and the Open-Window re-run, never the owner check's triggers" / dobby's sleep watch replaces the in-process one — **broken** for the owner trigger's own sleep watch and the owner fold's placement.
4. D4: "Bound: a pass stays in flight only through those waits" — **broken**.
5. D4: `--continue-pass` refused unless `running` at Run-Shed and named in the dobby state record with no full pass owed — **held** (an outside caller meeting the condition does what dobby would do).
6. D4: yield leaves status file, history and activity untouched — **held** for the status file.
7. D4: never two passes of one slug, held driver lock read as pass in flight — **held** (a second pass meets `driveActionBusyExit`, `internal/sheddriver/drive.go:244-248`).
8. D4: `battenAlive` change still reads a halted run as settled — **held** (the change applies only while `running`).
9. D1: `lyx reed down` spares dobby; reap/tender don't end it by accident — **held** (`Reap` never ends a process whose pair exists or a hub-owner process, `internal/tenderengine/reap.go:33`; `EndOwnedElsewhere` on a beam slug matches D1's "exits when owned elsewhere").
10. D1 bound: `runproc.End` of beam slug from its landing run's cancel/owner pause ends dobby only when the beam has no members — **held**.
11. D2 + acceptance: "`lyx adopt` ends with `lyx beam start`, so a dobby runs wherever the adopted runs' parent now is" — **broken**.
12. D5: "No code path but an explicit release leaves a task statusless" — **broken** (two ways).
13. D5: `settleClaim` writes `draft`; owned-elsewhere `lyx fabric remove` writes no board status — **held** (`Local`, `settles := !req.Local && !req.LeaveClaim`, `internal/pairteardown/teardown.go:369`).
14. D5: older binary's rewrite keeps `draft_status` key — **held** (`storeFile.Unknown` writes back unknown top-level keys, `internal/boardengine/store.go:63-80`).
15. D10: "a restart neither misses nor double counts" a landing — **broken**.
16. D12: "no run enters Land under a pause" — **mostly held**, narrow race (finding 10).
17. D4 respawn cap: "a write of the child's status file since the last respawn resets the count" — **not verified** (could not determine whether `lyx loom start` itself writes the child status file; if it does, every respawn would reset the count).
18. D13: ≤1 wakeup/s, backoff while idle — **held**.

**Findings**

1. **BLOCKING · design · D3/D4 (owner takeover after a wake or start)**
   A full pass after a takeover never reaches InnerRun's owned-elsewhere handling and loops. `reconcileResume` → `resumeRefusal` returns `ownedElsewhereRefusal` before the driver steps (`internal/battencli/resumereconcile.go:42-61, 80-81`), so the pass exits non-zero. D4 keeps an owed full pass "until a full pass of that run exits 0" and says "the retries go on", so dobby re-runs the refused full pass every probe tick indefinitely, posting after three a "failing pass" notice instead of the owner notice; `ownedElsewhere` (`internal/battenshed/innerrun_owner.go:101-117`) — `PauseOwned`, parent-lock release, adopt-naming notice — never runs. Nothing in D3/D4 tells dobby to stop driving a run once read owned elsewhere, so "drives only runs this machine owns" has no mechanism. `lyx orch start`'s `pauseOwnedRuns` (`internal/orchcli/start.go:283-300`) pauses such runs only on an orch start, and even then D4 keeps passing covered `running` runs, each owed full pass refused again.
2. **MEDIUM · design · D4 (spawn hold returns Yield)**
   With an owner standing, the yield skips the only re-check, so the hold never clears. In `holdSpawnWhileCovered` (`internal/battenshed/innerrun_pause.go:47-71`) the once-per-probe `ownerStep` sits after the `Sleep`; returning `Yield` "instead of sleeping" yields before it. With `ownedBy` persisted (D4's inventory), every later pass meets `mustSpawn`, enters the hold and yields at once; the `foldOwnerTrigger` retry arm sits in `check()`, which this path never reaches. Counterexample: run read owned by B, operator `lyx adopt`s back to this machine — every pass still yields "held: owned by machine B" forever.
3. **MEDIUM · design · D4 (owner trigger across passes)**
   The owner trigger's own sleep watch is not covered, and a one-check pass may never fold the owner trigger at all. `foldOwnerTrigger` reads `p.ownerSleep` against `deps.Awake`, which defaults to process-relative `time.Since(origin)` (`internal/battenshed/innerrun.go:264-266`), so in a fresh process per pass it can never see a sleep; D4's inventory lists `ownedBy/ownerNoticed/ownerDue/ownerCovered` but not `ownerSleep`, and replaces only "the idle checks' sleep watch" with dobby's sleeps. Further, `wait()`'s first round (`innerrun.go:669-678`) runs `coverage`, `noticeStep`, `runArm` but never `foldOwnerTrigger`/`ownerStep`; those run only in the loop's `check()` (`innerrun.go:700-710`). "Never the owner check's triggers" therefore holds only if D4 names which check its one-check pass runs and that it includes the owner fold — it names neither.
4. **MEDIUM · design · D2 / acceptance "the parent of a run `lyx adopt` took has a live dobby"**
   Adopting one member task of a beam owned on another machine leaves the run with no dobby. `lyx adopt <slug>` takes single task slugs (`internal/adoptcli/adopt.go:214-224`); the task's owner becomes this machine A while its parent beam stays owned by B. `lyx beam start` on A starts no dobby for a beam A does not own (D2), and B's dobby drives only runs on its own machine (D3), so the adopted run on A has no dobby and the acceptance criterion fails.
5. **MEDIUM · design · D5 "every board write that creates a task"**
   A note upserted to `kind: task` makes a statusless task D5 does not list. The board itself counts this as task creation ("A new task, a promote, a note upserted to kind task and a merge upserting a new task all count", `internal/boardengine/taskguard.go:19`), but D5 lists "an upsert, a batch upsert or a merge, and `lyx board promote`" — an upsert of an existing note to `kind: task` edits an existing entry rather than creating one, so it reads outside the list. The task comes out statusless and dobby starts it with no release, breaking "No code path but an explicit release leaves a task statusless."
6. **LOW · consistency · D4 "a pass stays in flight only through those waits"**
   Two in-pass waits are missing: Land's deploy under `deployTimeout = 30 * time.Minute` (`internal/battenshed/land.go:33, 334`) and InnerRun's spawn running `lyx loom start --no-attach` synchronously via `cmd.Run()` (`internal/battencli/wire.go:1205-1213`).
7. **LOW · consistency · D5 "explicit release"**
   A status can also be cleared outside the named release verb: the upsert payload accepts `status` (`internal/boardengine/store.go:448`), and merge's `set_status` takes a nil `Status` to clear (`internal/boardengine/store.go:436-441, 1074-1077`). D5 names only `lyx board set-status <slug>` as the release, leaving these two routes to an empty status unaccounted for.
8. **LOW · design · D10 "a restart neither misses nor double counts"**
   Counterexample: dobby down, a hand pass lands the run, then `lyx batten cleanup --apply` removes its run directory (`internal/battencli/cleanup.go:217`) before dobby restarts. The Land Done is gone from batten history and never counted.
9. **LOW · design · D12 "no run enters Land under a pause"**
   A pause written after InnerRun's coverage read but before the driver steps into Land is not seen: Land itself reads only batten's own pause flag (`internal/battenshed/land.go:367-374, 404-411`), never a tender pause entry. Narrow window, but the claim is stated without it.

**Tool calls**

1. Grep for `removeRunRecords` callers and `board.json` file structs — manifest lacked the run-record removal path and unknown-key handling of the store.
2. Read `internal/adoptcli/adopt.go` adopt request handling — to confirm adopt takes a single member task, not in the manifest.
3. Read `internal/boardengine/store.go:60-105` and `internal/battencli/cleanup.go:205-225` — to confirm unknown keys are written back and how cleanup removes landed run records.
4. Grep `internal/loomcli` for the status write in `loom start` — to test the respawn-cap reset; inconclusive.
