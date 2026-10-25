```yaml
lens: existing-paths
outcome: reported
```

**Paths traced, per decision**

- D3/D4 wake → full pass → `reconcileResume` → `resumeRefusal` `EntryOwnedElsewhere`: **breaks** (F1).
- D4 full pass with origin unreachable → `refuseResumeUnreachable` resumes on the local Bolt, exits 0: **breaks** the owed-full-pass guarantee (F2).
- D4 spawn hold yields → `holdSpawnWhileCovered`'s owner re-check per notice probe: **breaks** (F3).
- D4 one check per Call → `wait` pre-loop round vs `check` (owner fold, `ownerStep`, `lockStep`, `refreshWatched`): **breaks** (F4).
- D4 bound "stays in flight only through those waits" → InnerRun `Spawn` (`lyx loom start --no-attach`, `cmd.Run`) and Land `RunDeploy`: **breaks** (F5).
- D4 persisted triggers → `ownerSleep` in `foldOwnerTrigger`: survives (a fresh process's awake clock never fakes a sleep; dobby's owed full pass stands in — but see F1 for what that full pass does).
- D5 creating writes → note upserted to kind task (`UpsertTask` → `ApplyPatch`, counted as task-making by `checkTaskCreation`): **breaks** (F6).
- D5 → `settleClaim` `claimClear` → `draft`: survives (`draft` is not a run status, so a later settlement keeps it, `claimKeep`).
- D5 → `lyx fabric remove` ended/owned-elsewhere (`Local: true`, no board write): survives, status untouched.
- D5 → cancel `clearRunClaim` changed to `draft`: survives; cancel's `LeaveClaim` teardown leaves the claim to cancel.
- D5 → upsert `"status": null` and merge `MergeStatusUpdate` nil Status: survives as an explicit release, but D5 names only `set-status` as the release (folded into F6).
- D4 `battenAlive` amended → `settleClaim` `claimHeld`, `driverChecksSkipped`, `settledForSessionDown`: survives.
- D1 `lyx reed down` → `endPairProcesses` (prime returns early) → `EndBeforeSessionDown` sparing `RoleDobby`: survives; teardown and cancel still end the dobby through `endOwned`.
- D1 → `tenderengine.Reap`: survives (dobby under an existing beam pair or `HubOwner` is never an orphan).
- D1 → `EndOwnedElsewhere` (tender, orch start's `pauseOwnedRuns`) ending the slug's processes: survives for a beam owned elsewhere, which D1 has exit anyway.
- D10 counting from batten history → `lyx batten cleanup --apply` `removeRunRecords` on done runs: **breaks** "a restart neither misses" (F7).
- D12 lift pass → Go driver `holdPause` at the stop call sites: survives as D12 states.

**Findings**

**F1 — BLOCKING, design, D3/D4 (wake and owner)**
A full pass after a wake turns a takeover by another machine into a looping failing pass, not an owner hold. `reconcileResume` returns `ownedElsewhereRefusal` before the driver steps anything (`internal/battencli/resumereconcile.go:80`, `owner.go:192-211`), so InnerRun's `ownedElsewhere` never runs: no `PauseOwned`, no release of the parent lock, no owner notice (`internal/battenshed/innerrun_owner.go` `ownedElsewhere`). D4 then keeps the full pass owed and retries it every probe tick ("the retries go on"), posting only the three-failure notice. D3's "drives only runs this machine owns" has no mechanism that stops dobby driving such a run, and D4 does not exclude a tender-paused run from passes, so even after `lyx orch start`'s `pauseOwnedRuns` pauses it the failing full passes go on.

**F2 — MEDIUM, design, D4 (owed full pass)**
"Cleared only when a full pass exits 0" is spent without an origin verdict when origin is unreachable: `reconcileResume` with `report.Unreachable` Warns and resumes on the local Bolt (`resumereconcile.go:52-55`, `refuseResumeUnreachable`), and the pass exits 0. A wake with the network still down clears the owed full pass, and the takeover is never judged afterwards — the owner check's retry trigger needs `ownedBy` already set, which nothing sets here.

**F3 — BLOCKING, design, D4 (spawn hold yields)**
With the hold yielding, no pass ever re-runs the owner check that ends an owner hold. Today `holdSpawnWhileCovered` re-runs `ownerStep` once per `noticeProbe` while `ownedBy` stands (`internal/battenshed/innerrun_pause.go` loop, `lastProbe`/`ownerDue`), and that check is what ends the hold. D4 makes it yield "instead of sleeping"; each pass hits the hold before `wait`/`check` (`innerrun.go:414-417`), so `foldOwnerTrigger`'s retry never runs. The persisted `ownedBy` keeps a driverless child held forever, even after `lyx adopt` takes the run back.

**F4 — MEDIUM, design, D4 "one check per Call"**
D4 does not say which round is the pass's one check, and the obvious one runs no owner or lock step: `wait` runs coverage, `noticeStep`, `runArm` before its first Sleep (`innerrun.go:669-678`), while the owner fold, `ownerStep`, `lockStep` (the idle release of the parent lock) and `refreshWatched` run only in `check` after a Sleep (`innerrun.go:700-714`). A pass made from the pre-loop round alone never fires the persisted owner triggers and never releases an idle owner's lock, contradicting D4's inventory purposes and the matching acceptance criterion.

**F5 — LOW, consistency, D4 bound "a pass stays in flight only through those waits"**
Two in-pass waits are missing: InnerRun's `Spawn` runs `lyx loom start --no-attach` synchronously with `cmd.Run` (`internal/battencli/wire.go:1205-1213`), and Land's deploy runs up to `deployTimeout` = 30 minutes (`internal/battenshed/land.go:33`, `:334`). Both keep a pass in flight and so delay D12's lift pass.

**F6 — MEDIUM, design, D5 "every board write that creates a task" / "no code path but an explicit release"**
A note upserted to kind task becomes a statusless task outside D5's list: `UpsertTask` patches the existing note through `ApplyPatch` (`internal/boardengine/store.go:514-515`), and `checkTaskCreation` counts this write as making a task (`internal/boardengine/taskguard.go:19`), yet it is not "a create", an upsert of a new task, a batch, a merge or a promote. Dobby then starts it with no release. Also, an upsert with `"status": null` and a merge's nil `set_status` (`store.go:438`, `:1074`) clear a status, while D5 names only `lyx board set-status` as the release.

**F7 — LOW, design, D10 "a restart neither misses nor double counts"**
`lyx batten cleanup --apply` removes the records run directory of every done run, history included (`internal/battencli/cleanup.go:91`, `:217`, `:277`). A landing made while dobby is dead (or between its ticks) is lost from the tidy count if cleanup runs first.

**Tool calls**
1. Grep for run-directory removal callers — context lacked who deletes the batten history D10 reads.
2. Grep of `internal/battencli/cleanup.go` for its removal scope — context lacked whether cleanup removes landed runs' records.
