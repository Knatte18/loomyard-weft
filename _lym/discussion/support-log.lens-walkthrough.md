```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end against the code, using one concrete case per decision.

**Findings**

1. **BLOCKING, D1.** When the task worktree is lost and the local branches survive, D1 never re-adopts the pair, so it falls through to D2's failed stop.
   - Case: the task worktree directory was deleted or lost, but the slug's warp branch is still in the local repository, unmarked. That is the usual shape of a "worktree gone" run. Only `fabric remove` deletes the branches, and that is D2/D3's case.
   - D1 calls `Topology.Add` with Parent and Anchor only, "as the Worktree-Create row does". In `internal/fabricengine/add.go`, `Add` refuses an existing unmarked warp branch with `ErrBranchExists` unless `AdoptLocal` is set. Worktree-Create (`internal/battencli/wire.go`) does not set it.
   - So `Add` refuses, and D2 fails the run. That misses D1's purpose and the board's workaround: `lym fabric add` has `--adopt-local` (`internal/fabriccli/fabric.go`).
   - D1 has to state that the re-adoption passes `AdoptLocal` when the slug's warp branch exists locally, and bound that.
2. **MEDIUM, D1.** With `AdoptLocal`, a stale worktree registration blocks the re-adoption, and D1 says nothing about it.
   - `add.go` says an `AdoptLocal` add refuses a warp or records branch "checked out in another worktree".
   - A worktree directory deleted without `git worktree prune` stays registered, and its branch reads as checked out there.
   - D1 would then refuse into D2 for exactly the lost-directory case. D1 has to prune the slug's stale registrations before `Add`; `removeSibling` already prunes for the same reason.
3. **LOW, D2.** D2's "the status leaves `running`" holds only for a `running` status.
   - `battenPreStepUnmarked` (`internal/battencli/arm.go`) resumes `failed`, `blocked`, `paused` and `awaiting` statuses as well.
   - `WriteFailedStop` returns `NotRunning` and writes nothing for those. D2 does not say what a hand pass on a paused run with a gone worktree does.
   - This is harmless for dobby, which plans passes only for `running`. A hand pass would still exit non-zero with D2's error. The record could say the write applies to a `running` status only.
4. **LOW, D2.** `WriteFailedStop` does not commit the status ("the next step's persist commits the file"), and the pre-run has no next step.
   - The `failed` status stays uncommitted and unpushed in the prime's records. Dobby reads it locally, so the purpose is met on this machine, but origin and other machines see `running`.
   - D2 should say whether the pre-run commits and pushes the status. The status push is detached under `PATTERN-push-both-sides`.
5. **LOW, D4.** The record never names the cause of the 2026-11-02 "missing necessary objects" rejection.
   - A plain tag push normally sends its objects, so whether D4's branch push before the tag removes the rejection rests on a cause no one has found.
   - The retry covers the rest. That uncertainty belongs under Open risks, not stated as settled in D4's rationale.

**Per decision: case used, purpose met or not**
- **D1** (resume at Run-Shed, worktree directory lost, branches local): not met. See findings 1 and 2. With the branches only on origin and the weft branch not archived, it is met: `Add` adopts from origin, then the reconcile reads the branch level.
- **D2** (pair removed by `fabric remove`, archive tag covering the weft tip): met for a `running` status. The run lock is free at preflight, because `battenPreStepUnmarked` releases its probe, so `WriteFailedStop` is not Busy. The commit gap is finding 4.
- **D3** (Finalize stop, then `lym fabric remove`, then `lym batten cancel`): met. With the pair gone, the child status is not found, so `childReachedParent` is false unless batten stands at Land or Worktree-Teardown, and the cancel clears the record.
- **D4** (removal whose last detached push exited on the held lock): met for ordering. Under the exclusive teardown lock, the only other records push is the prime's, on a different branch. Finding 5 applies.
- **D5** (tackler judging a tag-push stop): met.
- **D6** (removal after D4's push, so origin equals the local tip): the lease matches and the branch is deleted. With origin moved by a later run, the lease fails and the branch is kept. Met.
- **D7** (`Add` while the former pair's push child holds the lock shared): `Add` waits, then the caller spawns the new push after `Add` returns, and the push takes the lock shared. Met. A push child of the slug spawned while `Add` holds the lock exits without pushing, but no pair exists yet, so nothing is lost.

**Tool calls**
- Grepped `add.go` for `AdoptLocal`/`ErrBranchExists` and the callers of `Add` and `DiscardTaskBranch`: my context had no record of `Add`'s refusal rule.
- Read `internal/shedengine/failedstop.go`: for `WriteFailedStop`'s lock, state precondition and commit behaviour, which my context lacked.
- Grepped battencli for run lock and preflight sites, plus fabric add's adopt flag: to see whether the pre-run holds the run lock.
- Read `arm.go` `battenPreStepUnmarked`: to confirm the run lock is released before `ownerPreflight` and which states resume.
