```yaml
lens: existing-paths
outcome: reported
```

Task: the existing-paths lens over the D1–D12 decision record, against the code that already runs.

**Paths traced, by decision:**
- **D6 / D8:** `pushParent` → `LandLocally(parentHandle.HeadSHA())` → `recordHeldLanding` → `runpusher.RecordLanding`. That call replaces the run's ledger landing and sets its tip to the parent head. For the leader, and for a woken member, the parent head is not that run's own landing commit. **Breaks.**
- **D6 / D8:** `recordLanding` → `Deps.RecordLanding(parentHandle.HeadSHA())`, the dobby's landed-commit record. It names the last commit of the prefix, not the run's own. **Breaks.**
- **D6:** `syncParent` → `ParentAheadByOwed` → `AheadByOnly`. This needs every commit past upstream to be a ledger tip. It holds only while each prefix commit stays a tip. It breaks once the path above overwrites a tip.
- **D6:** fallback records hold `heldByRefOrLanding`. A trailer-named landing commit on the parent past origin holds the records of a run not yet done. It covers the window between the move and the ledger write. **Survives.**
- **D6:** `pushledger.HeldRecordsCommit` → `heldCeiling`. Holding a join-time records commit that origin already has caps the push at its first parent. Later status commits stay held. **Survives.**
- **D6:** the parent pusher's per-landing done write, read against a member whose Finalize has not ended. `withoutFinalizeEntered` / `finalizeEntered` keeps that member from being read as ended. **Survives.**
- **D6:** `lym batten lock release` and the idle release (`idleReleaseOn` / `finalizeKeepsLock`) during the batch gate. The verify marker or the Finalize status keeps the lock, and D6's lock re-read before the move catches a release. **Survives.**
- **D6:** the leader judging a member's reconcile verdict "as reads alone". `ReconcileOrigin` has a `scope.JudgeOnly` path, so a read-only judgment exists. **Survives.**
- **D5:** `ParentHoldsTaskWork` → `parentHoldsTaskWorkAt` (`pair.code.CurrentSHA()`). D5 rebinds it to the task branch ref. **Survives**, if done in fabricengine for every caller.
- **D5:** `readLocalBranch` on a detached HEAD. `UpstreamSHA` returns `ErrNoUpstream`, so the state reads as upstream unset and "ahead". D5 covers only the pusher and `RefusedPushStands`. Three other readers misread while the worktree is detached: `TipsOnOrigin` in batten's `level` (`internal/battencli/wire.go:1835`) and the dobby's `landedPairStays`, and `RunCommitsAhead` in tender's `PushOwed`. The cost is a delay or a respawned push, no loss. **Partly breaks.**
- **D5:** the `Deps.TaskHead` readers (recorder head, `rebaseAfterParentMerge`, `judgeFixerChange`, `mergeParent`). The batch path and every read after the restore use the task ref. **Survives.**
- **D5:** the verified-tree record in the shared `VerifyDir`. It is keyed by tree and command, and `BaseCommand` keeps the base entry. **Survives.**
- **D5:** `premerge.clear` and the merge-in base file, beside the batch's `RemoveLeftovers` calls with an explicit base. **Survives.**
- **D2:** an entry with an outcome whose run is torn down or cancelled and never resumes. No path removes it. **Leaks** (harmless cache).
- **D7 / D8:** `failed` and `conflict` runs through `gateAndFix` → `runFixer` → `freeLock`. The kept outcome keeps the entry out of batches. **Survives.**
- **D9:** readers of `Config.FinalizeLostRaces`: only `Call`'s loop, which D1 removes. **Survives.**

**Findings:**

1. **BLOCKING, design.** `_lym/discussion/decision-record.md:121` (D6 "The leader then does its own run's remaining landing records") and `:156` (D8 `landed` "does its remaining landing records").
   - Summary: today's remaining landing records read the parent head as "the commit the landing put on the parent". `pushParent` calls `LandLocally(HeadSHA())`, `recordLanding` calls `RecordLanding(HeadSHA())`, and `RecordLanding` replaces the run's ledger landing by slug.
   - Effect: the leader, its landing commit first in the prefix, overwrites its own D6 ledger tip with the last prefix commit. A woken member overwrites its tip with whatever head the parent has then.
   - Consequence: the parent is then ahead of upstream by a commit that is no ledger tip. `AheadByOnly` then fails, and every later Finalize's `syncParent` halts Stuck "ahead of its upstream" until the parent's push. This is the state D6's rationale says never arises.
   - The dobby's landed-commit record also names the wrong commit.
   - The record is silent on these paths: it rebinds only the size and empty checks to the landing commit's first parent.
   - Fix: state in D6 and D8 that a batched run's `LandLocally` and `RecordLanding` take its own landing commit from the entry or mark, never the parent head. Also state that a ledger landing the leader already wrote is kept, not replaced.
   - Evidence: `internal/landingshed/finalize.go:964-970` and `:1048-1053`, `internal/runpusher/parent.go:719-725`, `internal/fabricengine/landing.go:207-218`, `internal/landingshed/finalize.go:739-749`.

2. **LOW, design.** `_lym/discussion/decision-record.md:85-90` (D5 readers list).
   - Summary: while the leader's worktree is detached, `readLocalBranch` reads its code side as upstream-unset and "ahead". Three readers outside the pusher are left unnamed:
     - `TipsOnOrigin` in batten's `level` and the dobby's `landedPairStays`;
     - `RunCommitsAhead` in tender's `PushOwed`, which spawns pair pushes.
   - They read "not level" or "push owed" until the restore.
   - Evidence: `internal/gitrepo/blobread.go:376-377`, `internal/fabricengine/pusherreads.go:46-50` and `:204-221`, `internal/battencli/wire.go:1835`, `internal/beamcli/dobbydeps.go:807`, `internal/tendercli/deps.go:143`.

3. **LOW, design.** `_lym/discussion/decision-record.md:54` (D2 "an entry with an outcome stays until its own run, resumed, reads it").
   - Summary: a run whose pair is cancelled or torn down never resumes. Its entry with a `failed`, `conflict` or `landed` outcome is removed by no path and stays in the queue directory indefinitely.
   - Evidence: D2's text itself. Teardown paths (`pairteardown`) name no queue.

Tool calls:
1. `AheadByOnly` in `internal/fabricengine/landing.go`: needed its rule for which commits may sit past upstream, which the manifest lacked.
2. `runReconciler` and `ReconcileOrigin` in `internal/fabricengine/originreconcile.go`: needed to check that a read-only reconcile (`JudgeOnly`) exists for D6.
3. Callers of `RunCommitsAhead`, `TipsOnOrigin` and `RefusedPushStands`: needed the other-process readers of a detached HEAD.
4. `dobbydeps.go:795-815`, `wire.go:1822-1845`, `tendercli/deps.go:135-148` and `gitrepo.UpstreamSHA`: needed what those readers do and what a detached HEAD returns.
