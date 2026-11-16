```yaml
lens: existing-paths
outcome: reported
```

Task: the existing-paths lens over `decision-record.md`. For each state D1 to D10 introduce or change, I traced the code paths that already write, reset or read that state.

**Paths traced, per decision**

- **D1 transition push.** `SpawnStatusPush` callers (`fabriccli/spawn.go`, `anchorchange.go:557`, `reconcile.go:330`, `teardownlock.go:169`): the teardown-lock respawn and the status spawn retire as stated, and the anchor change and the reconcile migration stay detached. Survives.
- **D1 `PushPairAnchored` teardown lock.** It is a try-lock that returns `ErrPairPushRunning` at once; D1 turns it into a wait. Stated, survives.
- **D1 absorbing push lock and pending flags.** `pendingflags.go` `coalesceFlagged` and `respawnForeignFlags`: the lock holder respawns single-side detached pushes for sides another pusher flagged. **Breaks** (F1).
- **D1 lock-busy outcome.** A wait for the absorbing push lock past `lockWait` returns `ErrPushLockBusy` with "neither side pushed" (`pushanchored.go:58-61`). It falls under D1's transient-failure owed push. Survives.
- **D1 pause deferral.** `coalescePushSides` installs a gate deferral, but `pauserecord.Network` returns before that deferral while an `Admit` admission stands (`hold.go:423-437`). Survives.
- **D2 Finalize order and `PastParentMerge`.** The act rule reads origin's parent branch (`act.go:29-30,91`), and D2's parent push before its transition keeps that read consistent. Survives.
- **D4 origin-owed marker.** Read by the dobby's pass inputs (`dobbyengine/passes.go`, `OriginOwed`) and by batten's continue-pass check; D4 names both. Survives.
- **D5 `hold` caller against the existing constants.** `pauserecord.HoldCaller` already means `"adopt-hold"` (`record.go:29-32`). **Clash** (F3).
- **D5 dobby `Held` field.** It reads only `AdoptCaller` and `HoldCaller` entries as hand-held (`passes.go:53-56`). After the upgrade's first read no such entry stands. Survives.
- **D5 `Coverage.Held`, `Coverage.OwnerHeld`, the gate's `OwnerEntryExempt` and the dobby owner lift (`beamcli/dobbydeps.go:567`).** All retire with the `owner` caller. Survives.
- **D5 hibernate wake `holdRuns`.** The owner branch retires as stated (`wake.go:110-118`). Its pauses of stopped runs and busy ended runs are slug entries, and D5's "a slug with a live entry gets no `hold` entry" keeps them apart. Survives.
- **D5 `tenderengine.Resume` and `Lift`.** A hub-wide resume lifts every live entry (`resume.go:139-145`); D5 states the exception for `hold` entries. Survives.
- **D5 `lym batten cancel`.** It uses `Aside.Hold` and `SetGateExemptAll` (`cancelcmd.go:202,460`), so it goes on under a hold, as D5 states. Survives.
- **D6 adopt record readers.** Its importers are `orchcli/watch.go` (named in D5), the `adoptcli` files (retired) and `beamcli/cli.go:397` with `beamcli/adoptsection.go`. **Breaks** (F2).
- **D8 printed `parent_beam` readers.** `contracts/orchrole/orch-role.md`, `plugins/ly/skills/board/SKILL.md` and `boardcli` doc and help are all named in D8. `boardcli/runsettings.go:128` reads `ParentBeamOf`, which D8 leaves unchanged. Survives.
- **D9 `orchcli.BeamDown` callers.** Named in D9. Survives.

**Findings**

F1. MEDIUM, design, D1 (bullets 3 and 5) and D2 (bullet 2).
- **Summary:** D1 retires only the teardown-lock respawn. The pending-flag respawn survives: on release, the absorbing push lock's holder spawns a detached push of each side another pusher flagged, one side at a time. So a code side can still land alone, outside the records-first order and outside a takeover's "push no code side".
- **Counterexample:** The old owner's synchronous transition push marks its code and records flags before taking the lock. It meets a holder that owes only the records side of the same records worktree, and its lock wait gives up, which leaves the push owed. On release, the holder respawns the flagged code side alone. Once a takeover has landed, that fast-forward lands the old owner's code tip on origin, which breaks D2's criterion that "the old owner's next transition and its `lym loom commit-records` leave origin's code branch unchanged".
- **Fix direction:** D1 should state what becomes of the pending-flag respawn: it retires for pair sides, or it respawns through the records-first pair push with the takeover read.
- **Evidence:**
  - `internal/fabricengine/pendingflags.go:120-155` (`coalesceFlagged`, "a flag of a side it does not push is respawned as a detached push of that side")
  - `pendingflags.go:194` (`respawnForeignFlags` "spawns that side's push from it")
  - `internal/fabricengine/pushanchored.go:80-81` ("each side it pushes is marked pending before the lock is taken… pushed by whichever holder releases last")
  - `internal/fabricengine/coalesce.go:91-96`

F2. MEDIUM, design, D6 ("Retired: … the adopt record") and D5 (last bullet).
- **Summary:** D6 retires the adopt record and moves only the dobby's handover and the orch watcher onto the hold. `lym beam status` also renders its `adopt` section, with subjects, step, state and the re-run way forward, from the adopt record. The record is silent on what replaces that section. A standing Bolt hold blocks every run on every machine, yet the status view the operator reads names neither the hold, its machine, nor this machine's `lym adopt &lt;subjects&gt;` re-run; D10 prints that only in `lym machine start`.
- **Evidence:** `internal/beamcli/cli.go:397-401` (`adoptrecord.Read` feeding `result.Adopt`), `internal/beamcli/adoptsection.go:1-40`.

F3. LOW, consistency, D5 (bullets 4 and 6).
- **Summary:** The new pause caller is literally `hold`, but `pauserecord.HoldCaller` already names the legacy `"adopt-hold"` caller. D5 keeps that constant "declared only for the upgrade's first read", so the two coexist. The record does not say what the new caller is called in code, and readers that test `HoldCaller` keep matching `adopt-hold`, not the new `hold` entries.
- **Evidence:** `internal/pauserecord/record.go:29-32`, `internal/pauserecord/doc.go:22-24`, `internal/dobbyengine/passes.go:53-56`.

**Tool calls**

1. `coalesce.go`, `pendingflags.go` and the `SpawnStatusPush` callers: the coalescing and respawn paths a synchronous push meets were outside the manifest.
2. Repo-wide `parent_beam` grep and the `adoptrecord` importers: the readers of D8's printed value and of D6's retired record were outside the manifest.
3. `dobbyengine/passes.go`, `pauserecord` coverage fields and `hibernateengine/wake.go` `holdRuns`: the existing readers and writers of the pause callers D5 changes.
4. `pushanchored.go` body: the side order, pending flags and lock outcomes D1 changes; the manifest held only the symbol's location.
5. `beamcli/adoptsection.go` and its call in `beamcli/cli.go`: how `lym beam status` reads the adopt record D6 retires.
