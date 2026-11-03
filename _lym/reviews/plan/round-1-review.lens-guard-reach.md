```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch and test row the cards add, I traced whether a real entry point can reach it and whether the guarded site recurs on a path no card guards.

**Per guard, reachable or not**

- Card 1:
  - The "missing necessary objects" retry row is reachable.
  - The leased delete's `stale info` → `ErrLeaseRejected` mapping is reachable, through a moved origin branch.
- Card 2:
  - `PushBranch`'s rejection is reachable through a diverged origin.
  - The marked-branch `ErrLocalBranch` refusal is reachable through its own test. No `Remove` path reaches it, because `archiveWeftTip` returns early on a marked branch. As a primitive's contract it is no finding.
- Card 3:
  - The drop for a forked or locally adopted weft whose origin copy is absent is reachable, and so is the drop for a forked warp branch.
  - The `config.lock` failure row is **not reachable as written** (finding 5).
- Card 4:
  - Weft not live (`ErrNotLive`): reachable.
  - Task branch gone locally and on origin (`ErrNotLive`): reachable.
  - `ErrRemoteLeftover` under adopt-only, for a diverged local weft: reachable. The warp leftover cannot fire there, because `probeWarpLeftover` with `live` set never refuses (remoteleftover.go:168).
  - `ErrRecordsWorktreeNotPair`: reachable.
  - `ErrTaskPathTaken`: reachable. Card 10's `taskWorktreeGone` reads a stray path as gone, and Add's path check at add.go:298 comes before the probes.
  - `ErrAdoptLocalCheckedOut` for the task branch: reachable.
  - `ErrAdoptLocalCheckedOut` for the records branch: reachable only when the warp branch is local. The same site recurs unguarded (finding 1).
  - Reusing a standing records worktree: reachable. A fast-forward over it is unguarded (finding 2).
- Card 5:
  - Add's wait-out refusal: reachable.
  - The discard's wait-out refusal: reachable.
  - `PushPairAnchored` skipping its push under an exclusive hold: reachable, through commit-records, Finalize and `lym fabric push`.
  - The same guard and test row on `PushAnchored`: reached by no production entry point (finding 6).
  - I found no other synchronous push of a pair's weft branch outside the lock. `Fabric.PushWeft` has no production caller, and `Fabric.PushBranch` pushes the warp side.
- Card 6:
  - The branch push when origin lacks the local tip: reachable.
  - A failed branch push that is only logged: reachable.
  - The second tag for a diverged or ahead origin tip: the step-2 ancestry test errors before the step-3 fetch (finding 3).
  - The retry through `archiveTagPushHook`: reachable.
- Card 7:
  - Lease refused, branch kept: reachable through the hook.
  - Empty lease, no delete: reachable when origin has no weft branch and the branch push failed.
  - The origin-only path's leased delete: reachable.
  - No other deletion of a weft branch on origin goes unleased; Add's step 11 is already leased, and `rollbackAdd` passes `remote` false.
- Card 8:
  - Held-lock skip: reachable.
  - Lock-directory error: reachable through a file at `teardown/`.
  - The remote warp sweep's held-lock keep: reachable.
  - What the lock is meant to exclude is read before the lock is taken (finding 4).
- Card 9:
  - `PairMayBeAbsent` with an unregistered task worktree: reachable. `originTargets` reads registration through `List` (originreconcile.go:632).
  - Without the option: today's error, reachable.
- Card 10:
  - The gone path: reachable. The pre-run runs `refuseCovered` and `checkRunStatus` and then `preflightRunOwner`, and none of them stats the task worktree (arm.go:720-735).
  - `refuseContinuePass` with `Gone`: reachable; it runs first under `--continue-pass` (arm.go:720-722).
  - Unreachable-origin refusal: reachable.
  - Worktree-Create and Worktree-Teardown rows skipping the readopt seams: reachable.
- Card 12:
  - The permanent set: reachable, apart from the records-branch case in finding 1.
  - Busy from the run lock: reachable. `WriteFailedStop` tries the lock once and reports Busy (failedstop.go:50-55).
  - A `failed` status re-run keeping its state: reachable through `checkRunStatus`'s resume of a failed status (arm.go:586).
  - Dobby planning no pass for a `failed` run: reachable (passes.go:97).
- Card 13:
  - Owner-held, `LeaveClaim`, a record at Land and a record before Land: all reachable through `lym fabric remove`.
  - The tender ender's appended follow-up: reachable.

**Findings**

1. MEDIUM, design, card 4 (`04-add-adopt-only.md`): D2's permanent refusal "records branch checked out in another records worktree" fires only when the warp branch is local.
   - `refuseAdoptLocalCheckedOut` runs only under `opts.AdoptLocal &amp;&amp; warpAdopted` (add.go:341). Card 4 keeps that call site.
   - An adopt-only re-adopt that takes the task branch from origin, with the local weft branch checked out in another records worktree, skips the check. It then fails at `containedWorktreeAdd` with a plain `adopt weft worktree … failed` error (add.go:493-498).
   - Under card 12 that error is transient, so dobby retries the pass every time: the endless failing pass the task exists to end.
   - Evidence: add.go:341-345, add.go:493-498, decision record D2 bullet 4.
2. MEDIUM, design, card 4: reusing a standing records worktree leaves Add's fast-forward fetch unguarded.
   - When the local weft branch is behind origin, `weftOld.fastForwardTo` is set (remoteleftover.go:151-152). Add then fetches into `refs/heads/&lt;weft&gt;` (add.go:468-473).
   - git refuses to fetch into a branch checked out in any worktree, and the reused records worktree holds that branch. The re-adopt therefore fails with a transient error on every pass.
   - Card 4 handles the reuse at the directory refusal, the worktree add and the rollback, but not at this step.
   - Evidence: add.go:468-473, remoteleftover.go:131-154.
3. MEDIUM, design, card 6 (`06-remove-archive-order.md`): the step-2 test of whether origin's tip descends from the local tip cannot be answered when origin's tip is not in the local store.
   - Step 2 decides whether origin lacks the local tip ("not equal … and not a descendant") before step 3 fetches origin's tip by SHA.
   - `isAncestorOrEqual(dir, local, originTip)` runs `merge-base --is-ancestor` against a commit the records repository lacks. That exits 128 and is returned as an error (ancestry.go:33-41, remoteleftover.go:221-229), so the archive, and the removal with it, fails.
   - This is exactly the case of the card's "diverged" and "ahead" test rows, where another clone moved origin. The second-tag branch is not reachable as ordered. Fetching origin's tip by SHA in step 1, before step 2, would fix it.
   - Evidence: ancestry.go:25-43, remoteleftover.go:221-230, card 6 steps 1–3.
4. MEDIUM, design, card 8 (`08-cleanup-teardown-lock.md`): the per-slug try lock is taken after the sweep has already read the state it is meant to guard.
   - Cleanup reads `liveWarpBranches` and `listWeftBranches` before its loop (cleanup.go:131-183). `cleanupRemoteWarp` reads the candidates, the `checkedOut` map and the tips before deleting anything (cleanupremotewarp.go:67-112).
   - An `Add` of the slug that completes between enumeration and the try is not seen. In the remote warp sweep it can then lose the origin copy of a task branch a re-adopted pair just took over, for example a landed task branch at a D1 Land re-adopt. The lease still matches, because the new pair has not pushed yet.
   - In the weft `Cleanup` it is only a spurious entry error: the local delete's checked-out gate refuses (cleanup.go:341-357).
   - Card 8 claims that "no new pair's Add interleaves with a deletion of the slug's former task branch", but re-reads nothing under the lock.
   - Evidence: cleanupremotewarp.go:89-112, cleanup.go:164-221.
5. LOW, consistency, card 3 (`03-add-drops-stale-tracking.md`): the `config.lock` test row reaches its refusal only if the upstream config stands.
   - `DropRemoteTracking` touches config only when `branch.&lt;b&gt;.remote` or `branch.&lt;b&gt;.merge` is set (card 2). Deleting a ref needs no config lock.
   - The row states only "a stale weft tracking ref standing", so the drop succeeds and the expected error never comes. Add's only earlier config write, `disableStoreAutoGC`, is non-fatal (add.go:407-409).
   - The row must also set the weft branch's upstream config.
6. LOW, consistency, card 5 (`05-teardown-lock-add-discard-push.md`): `PushAnchored` has no production caller.
   - The card's claim that commit-records and `lym fabric push` run `PushAnchored` does not hold. Production callers are only of `PushPairAnchored`: loomcli/wiring.go:214, loomcli/landingdeps.go:329 and fabriccli/weft_verbs.go:295. A search found no caller of `PushAnchored` outside pushanchored.go and doc.go.
   - The lock guard and the `PushAnchored` test rows on it guard a path no entry point reaches.

Tool calls:
1. `gitrepo.IsAncestor` and the batten pre-run order in arm.go: the context lacked how a missing commit is handled and what runs before `preflightRunOwner`.
2. arm.go `battenPreRun`, `WriteFailedStop`'s locking, and the other `PushCoalesced`/`PushRebaseFree` sites: needed for the reach of the gone path, the Busy refusal and any unguarded weft push.
3. recordsgit.go `PushWeft` and spawn.go `PushWarpAt`: needed to judge whether those push sites push a pair's weft branch synchronously.
4. Callers of `Fabric.PushWeft` and `Fabric.PushBranch`: needed to confirm whether either push site has a production caller.
