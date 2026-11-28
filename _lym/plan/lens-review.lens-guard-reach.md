```yaml
lens: guard-reach
outcome: reported
```

Guard-reach review: for each guard the cards add, whether a real entry point can reach its state and whether the guarded site recurs unguarded.

**Findings**

1. **BLOCKING** (card 8 `judgeMember`, D6 "judges them as reads alone and acts on no member's behalf"): the member check calls `reconcileReport` over the member's own `Deps.Reconcile`. That seam is not a read, so the "action needed, leave the member out" guard sits after the action has already run.
   - `internal/loomcli#runReconciler` (landingdeps.go:796-808) runs `fabricengine.ReconcileOrigin` with `pairteardown` origin seams.
   - `internal/landingshed#Finalize.landUnderLock` (finalize.go:470-472) says "The owner checks' run-scope reconcile can fast-forward the task branch".
   - So the leader can fast-forward the member's task branch, or run teardown seams on its behalf. That is the exact action D6 keeps with the member's own Finalize.
   - Fix: give the member a read-only reconcile seam, such as a reconcile with no fix-up seams, or read only the board's `owned_elsewhere`/ended state, and state it on card 8.

2. **MEDIUM** (card 7 `mergeNeedsSession` and card 3 `Fabric.MergesClean`, D8 last bullet): `Deps.MergesClean(ParentBranch)` passes a branch name. Card 3 says `MergesClean` runs `MergeTree(base, source)` but never says it resolves refs.
   - `internal/gitrepo#Repo.MergeTree` returns `ErrInvalidSHA` for a non-SHA argument. Without resolution, every queue-mode merge-in under the lock errors at this guard.
   - Only tier0 fake rows reach the guard, so no planned row would catch it.
   - `Fabric.HoldsWork` resolves refs with `ResolveSHA` first (parenttips.go:225). Card 3 should state that `MergesClean` does the same, and a tier1 row should reach it.

3. **MEDIUM** (cards 7 and 8, D10 "a run known before the lock to need GitHub writes no entry and lands alone"): no guard keeps a run with no entry from leading a batch.
   - Card 7's `joinQueue` writes no entry for a run whose `needsGitHubOnDisk` reads true or fails. Under the lock its `actOnOutcome` finds no entry and picks "none: the run leads".
   - Card 8's `takeBatch` starts from "the run's own entry first" and only refuses a leader whose own `pullRequestLanding` sends it through a pull request.
   - So a run with no entry whose decision under the lock is a local landing (for example after a failed on-disk read) still takes other members. It should land alone.
   - Fix: card 8 adds "a run with no queue entry leads no batch", and a row that reaches it.

4. **LOW** (card 7 `actOnOutcome`, the "`landed` with commit on parent and task head equal → held" branch): it is reachable only when `parentHoldsWork` reads false and that same commit is still on the parent's head.
   - `parentHoldsWork` is read first. `Fabric.HoldsWork` judges by tree equality, so a parent holding the landing commit and the same task head reads held already.
   - The branch is live only after a later parent commit undid the work. No planned row reaches that state, and the card names none.

5. **LOW** (card 10 `Finalize.detachedStop`, "a detached clean one with a marker, reached with no shed restore before it, is restored"): no production path reaches this state.
   - `Shed.Restore` is set in `loomCLI.newShed` (arm.go:302), the only builder of loom sheds through `loomrecipe.New`. Card 10 runs it in both `Run` and `Step` before any producer.
   - So Finalize never meets a clean, marked, detached worktree.
   - The card names no input and no row for it. Drop it, or write it as a fail-loud wiring guard.

6. **LOW** (card 4 `parentHoldsTaskWorkAt`, "a marker that names no existing branch is an error naming the marker"): the card names no input and no test that reaches this error.

7. **LOW** (card 11, "an error releases the lock and is returned, naming the queue directory"): reachable through a ledger write failure, but `TestAcquireBranchLock_WritesTheLandingsAStandingMarkOwes` has no row for it.

**Guards checked, no defect**

- The backstop's coverage of lock takers holds: only `fabriccli#acquireBranchLock` (anchordeps.go:567) and `landingshed#parentGuard.ensure` (parentguard.go:78) call `Store.Acquire`, and no `TryAcquire` or `HandOff` caller exists outside `parentlock`. So cards 7 and 11 cover every taker.
- The pusher hold-off recurs at every `readLocalBranch` caller (`RunCommitsAhead`, `TipsOnOrigin`, pushanchored.go:132) through card 4's one edit.

**Tool calls**

- Shed construction sites: to confirm `loomCLI.newShed` exists and is the only loom shed builder, for finding 5.
- Parent lock store construction and acquire sites: to list every taker of the lock.
- `runReconciler` body: to confirm the member reconcile is not read-only, for finding 1.
- Other `Acquire`/`TryAcquire`/`HandOff` calls: to rule out further takers.
