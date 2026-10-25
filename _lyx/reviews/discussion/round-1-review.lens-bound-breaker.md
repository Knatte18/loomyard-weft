```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every bound, guarantee and claim about existing code in the discussion record.

**Claims attacked**
- D4: "between those, InnerRun's own owner check covers it" (continue passes skip the origin reconcile). **Broken.**
- D3: "the owner check inside each pass covers a run another machine takes over". **Broken**, same cause as the D4 owner claim.
- D4: InnerRun memory keeps the "once-per-episode, backoff, idle-bound and grace rules" across passes. **Broken as stated**: the memory that rule depends on is not named.
- D4: "a pass's own writes never trigger the next", and the per-run pass bound. **Broken** by events delivered after a pass exits.
- D4: respawn cap of three per child state episode, while "each pass ... respawns a driverless running child". **Broken** over a long running episode.
- D4/D12: a pass is one check of the child that yields, and "batten's existing pause handling holds every spawn". **Broken**: the pause hold and Land's waits are long in-process waits.
- D4: "Shed persists nothing on a yield". Held: activity is composed in the persist that follows an outcome (`internal/shedengine/activity.go:18`).
- D4: a held `shedrun.DriverLock` is read as a pass in flight. Held: batten's pass holds that lock for the driver's life (`internal/battencli/drive.go:45,65`).
- D5: "a task is started only from an empty status", with cancel leaving `draft`. **Broken** by the pair-teardown claim clear.
- D5: the migration "never runs again" and an older binary keeps the key. Held: unknown top-level keys are kept (`internal/boardengine/store.go:62-70,283-289`).
- D5: tasks an older binary makes wait for a release. **Broken** for tasks made after the key exists. The support log rejected this at LOW.
- D1 bound: only a cancel or owner pause of the beam's landing run ends a beam's dobby, by which time the beam has no members. **Broken** by `lyx reed down`.
- D1: a second dobby exits on the held lock. Held.
- D8: the arbiter moves into a shed-layer package batten imports without tree-sitter. Held: `battencli` already imports `loomengine`, and `quarry/glyph` has no cgo files.
- D8: a `loom.yaml` value set in a hub layer "keeps its meaning ... an explicit 0 included". Held for `arbiter_max_spawns`. Weak for `arbiter_timeout_min`, and darn's copy covers only the hub layer (LOW).
- D10: each landing is counted once across a restart. **Broken** for landings made by a hand pass or by a pass whose dobby died.
- D13: wakes at most once a second and backs off while idle. Held.
- Constraint: batten's "notice markers already survive a restart". Partly broken: several notice latches are in-process booleans. Folded into F3.

**Findings**

**F1. BLOCKING · design · D4 continue pass, and D3.** Skipping the origin reconcile relies on an owner check that never fires inside a short pass.
- Evidence: `internal/battenshed/innerrun_owner.go:67-75` sets `ownerDue` only when this process's sleep watch sees a sleep, a pause covering the slug is lifted, or a retry is due while the run is already owned elsewhere. `ownerSleep`, `ownerDue` and `ownerCovered` are fields of the producer in memory (`innerrun.go:165-170`).
- Counterexample: dobby keeps running while the laptop sleeps, and another machine runs `lyx adopt` on the run. On wake the next pass is a continue pass, so there is no reconcile. Its new process saw no sleep and no pause lift, so no owner check runs. The pass respawns or lands the run on the old machine.
- Rule broken: `PATTERN-owner-machine` ("acting on an entry after a wake: origin reconcile first"). Neither D4 nor D13 says dobby detects a wake, or that a wake makes the next pass a full one.

**F2. BLOCKING · design · D5.** A pair-teardown claim clear empties the status, and dobby then restarts the task.
- Evidence: `internal/pairteardown/claim.go:45-50` returns `claimClear` for a batten run that was never seeded or is paused, blocked, failed or awaiting. `writeBoardClaim` (`claim.go:166-172`) then sets a nil status.
- Counterexample: the operator runs `lyx fabric remove` on a pair whose batten run halted. The claim is cleared, the status is empty, dependencies are done and the parent is the dobby's worktree. The next probe starts a fresh run of a task the operator just removed.
- D5 moves only `lyx batten cancel` to `draft`. Its bound "a task is started only from an empty status" relies on no other writer producing an empty status, and `pairteardown` is one.

**F3. BLOCKING · design · D4 memory record, and Open risks.** The record does not name the state the per-episode rules depend on, and some of it is per Call today, not per producer.
- Producer fields kept in memory: `driverBusyNoticed`, `ownerNoticed`, `ownedBy`, `episode.deliveries` (delivery attempts, `lastTry`, `strandWarned`, `dropped`), `episode.arbiterOffAt`, the `failures` episode set, `restep` (interval, `nextAt`, attempts) and the owner fields. Evidence: `innerrun.go:140-170`, `notice.go:215-238`, `innerrun_binaryrestep.go:34-46`.
- Per-Call state: `lockWatch.idle`, which is the idle bound itself, plus `resumeRetryAt`, `notParkedLogged`, `busyLogged`, `busyKey` and `sleep`. Evidence: `innerrun.go:554-600`, `innerrun_parentlock.go:14-19`.
- Counterexample: the parent-lock release waits until the owner has idled for `IdleBound` (`innerrun_parentlock.go:25`). Each pass runs one check in a new process and starts a new `IdleWatch`, so the bound is never reached and the lock is never released.
- The record says "every field" and leaves the inventory to an implementer. That is a deferred research question under the rubric. The Constraint that notice markers "already survive a restart" is also only partly true: the busy-pane and owner-elsewhere latches are in-memory booleans.

**F4. MEDIUM · design · D4 respawn cap.** A cap of three per child state episode stops respawns of a driver that dies late in a long running episode.
- Evidence: the spawn confirmation is per pid (`innerrun.go:38-44`), and `driverAtWork` reads only the strand and the two locks (`innerrun.go:520-545`). Under passes, every pass's confirmation is unconfirmed.
- Counterexample: the child stays `running` for hours. Three passes land in the gaps between the child driver's hand-overs or relaunches, when no lock is held and the strand reads retiring or absent, so they respawn harmlessly and use up the cap. When the driver later truly dies in the same episode, the pass only sends driver-dead, which contradicts "respawns a driverless running child". The cap needs an episode key other than the child's state alone, or a reset when a live driver is seen.

**F5. MEDIUM · design · D4 and D12.** A "pass" can stay in flight for a long time, so "one check per Call", "yields" and the lift-time pass do not hold as written.
- Evidence: `holdSpawnWhileCovered` loops in-process for as long as a pause covers the slug or the run is owned elsewhere (`innerrun_pause.go:51-70`). The Land row sleeps in its own polls (`land.go:377,414`). The prime-lock wait runs up to 10 minutes (`primelockwait.go:14-19`). Row notices sleep 30 seconds between attempts (`rownotice.go:22,215`).
- Counterexample: a pause covers the slug while InnerRun must respawn. That pass blocks for the whole pause. The lift-time pass D12 promises is then skipped as "in flight", and the record names no bound on how long a pass may run. D12's "holds every spawn" keeps the in-process hold, which contradicts D4's "one check of its child per Call".

**F6. MEDIUM · design · D1 bound.** `lyx reed down` of a beam's session ends the beam's dobby while the beam still has runs.
- Evidence: `EndBeforeSessionDown` (`internal/pairteardown/skip.go:3,60-90`) ends every process registered under the pair's slug. Dobby registers under the beam's slug (D1). For the beam's own slug, `settledForSessionDown` (`skip.go:93-118`) reads only the beam entry's own batten and driver state, which is quiet.
- Counterexample: the operator runs `lyx reed down` on the beam pair's session. Dobby ends, and its member runs get no passes until someone runs `lyx beam start`. D1's bound names only a cancel or owner pause of the beam's landing run.

**F7. LOW · design · D4 self-trigger guard.** fswatch events can arrive after the pass that caused them has exited.
- D13 coalesces events into ticks of at most one a second. A batten status write made at a pass's end, such as a row Done or a halt, can be delivered after that pass's process exits. It then reads as "no pass in flight" and triggers one more pass.
- This is bounded to one extra pass, because a yield writes nothing. It still breaks D4's per-run bound of one pass per `notice_probe_s` plus one per child status write. Evidence: D4 and D13 text. The decision should key on the pass's exit time.

**F8. LOW · design · D10 landing count.** Landings dobby did not drive are never counted.
- Dobby counts "at the end of a pass whose batten history holds Land Done". A landing made by a hand-run `lyx batten run` pass is never seen at the end of a dobby pass. So is one made by a pass a dead dobby left running, which runs in its own process group.
- So "counted once per landing across a restart" undercounts. Basing the count on the pass that ends at Land Done, not on batten history scanned for all runs, is the gap.

**F9. LOW · design · D5 migration.** An older binary on another machine still creates statusless tasks after the migration key exists.
- Evidence: `store.go:62-70` keeps the key. An older binary's upsert still writes a task with no status, and an entry with no owner reads as owned by no other machine, so the owner machine's parent dobby starts it unreleased.
- The support log rejects this because "an owner machine runs its deployed binary", but the writer is on a different machine from the starter. The record should state this as an accepted bound.

**F10. LOW · design · D8 config migration.** The copy into `darn.yaml` covers only the hub layer, and one existing 0 changes meaning.
- Darn's copy names only "a hub layer". A loom arbiter key set in a worktree layer, or in a task's `run_settings.loom`, which darn reads today, is not copied, so darn's behaviour changes there.
- An explicit `arbiter_timeout_min: 0` in `loom.yaml` today means 30 minutes (`internal/loomengine/template.yaml:37`). Under D8 it inherits `shed.yaml`, which differs once `shed.yaml` sets another value. That contradicts "keeps its meaning".

**Tool calls**
1. Grep of `battencli` for `DriverLock`/`DriveVerb`, and of `shedengine` for activity writes: to check that a pass holds the driver lock and when a yield would persist activity.
2. `go list -deps ./internal/loomengine`, plus the arbiter file imports: to test the claim that batten can import the arbiter without tree-sitter.
3. `go list -deps ./internal/battencli` and glyph's cgo files, plus `loomengine` use in `arbiter.go`: the same claim, one hop further.
4. Grep of `shedengine/shed.go` and `activity.go` for persist points: to test "Shed persists nothing on a yield".
