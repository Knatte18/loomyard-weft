```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: traced the code paths that also write or read the state each decision changes.

**Findings**

1. **BLOCKING — D1: `Add`'s own refusals send the common gone-worktree case to D2, and the record doesn't say so.**
   - `Topology.Add` refuses when the warp branch exists locally and unmarked (`ErrBranchExists`), unless `AdoptLocal` is set. D1 passes no `AdoptLocal`.
   - `Add` also refuses when the weft worktree directory already exists (`internal/fabricengine/add.go`, "weft worktree directory already exists").
   - A task worktree that vanished any way other than `lym fabric remove` (a hand removal, a crash mid-teardown) usually leaves the local warp branch and often the weft worktree behind. D1 therefore re-adopts only when both local branches and the sibling worktree are gone too, and every other case lands in D2's failed stop.
   - The board expects re-adoption "when they still exist". D1 should state the `AdoptLocal` choice and how it treats a surviving weft worktree. Otherwise its bound ("only a live pair") reads wider than the code delivers.

2. **BLOCKING — D2: `shedengine.WriteFailedStop` neither commits nor can be sure to write.**
   - Its doc: "It runs no CommitStatus: the next step's persist commits the file." D2 is built so that no next step runs, because dobby stops planning. The `failed` status therefore stays uncommitted in the prime's records worktree, and origin and the board still read the run as `running`.
   - It also "takes the run lock without waiting and reports Busy when another holder has it". If the `lym batten run` pre-run already holds the batten run lock, the call reads Busy and writes nothing.
   - D2 needs to name its commit and push of the status, and the lock it runs under.

3. **MEDIUM — D6: a lease on the local tip leaves a newer origin weft branch, which the next `Add` then adopts.**
   - Case: a remover's local weft branch is behind origin. Cancel runs a judge-only reconcile, so it never fast-forwards, and the other machine's pushes or a skipped fast-forward leave the local copy behind.
   - D4's branch push is refused as not fast-forward, and the tag archives only the local tip. D6's leased delete then fails, so origin keeps the branch at a tip no `archive/<slug>/*` tag covers.
   - By `Add`'s liveness rule (`remoteleftover.go`) that branch is a live pair, so the next run of the slug adopts the cancelled run's records.
   - D6's bound claims such a branch is "the later run's live branch or one `Add` treats as a leftover". That is false here, because `Add` adopts it. The unleased delete used to remove it.

4. **MEDIUM — D4: the new synchronous weft push passes the pause gate, and the record doesn't mention it.**
   - `internal/gitrepo/remoterefslock.go` and `internal/gitexec/gitexec.go` both carry the pause gate. Per `PATTERN-remote-refs-lock`, a process passes the gate before it takes the lock.
   - A `Remove` that runs while a pause covers the slug or hub now blocks in its archive step while holding the teardown lock exclusive. Examples: `lym fabric remove` of a paused pair, the hub reconcile's teardown of an ended run, a Worktree-Teardown resume.
   - Every detached push of the slug and, under D7, every `Add` of it wait behind that block. Only an exempt cancel escapes the gate.
   - The record should state whether D4's push skips under a pause, which is what `DetachedPush` does today, or waits.

5. **LOW — D4: the push path the Notes suggest also pushes the code branch.**
   - The plan-writer note suggests reusing the coalesced records push. By `PATTERN-push-both-sides`, in a task pair that push carries the code branch along.
   - At a cancel's teardown it would publish the unlanded code branch just before the discard deletes it. D4 says "weft branch" only, so the note contradicts it. Fix the note, or state that the code branch is excluded.

6. **LOW — D1: a pair that disappears during a continue pass is not covered.**
   - D1 hooks only the resume path, since continue passes skip the reconcile. If the pair disappears while dobby drives continue passes, those passes fail some other way.
   - The record doesn't say whether that failure turns into a full pass, the resume where D1 then applies. If it doesn't, the bug-1 loop comes back through continue passes.

**Paths traced**

- D1:
  - Worktree-Create's `Add` → `reconcileTaskPair` → `spawnPairPush` sequence (`internal/battencli/wire.go`): survives. The reconcile runs before the pair push, as D1's order does.
  - `Add`'s pre-flight refusals: break the case (finding 1).
  - `adoptcli`'s `Add`: survives.
  - The continue pass: unclear (finding 6).
- D2:
  - `WriteFailedStop` lock and commit semantics: break (finding 2).
  - Dobby's `planPass` on a non-`running` status: survives.
- D3:
  - `finalize_owner.go`'s stop text: survives.
  - `lym fabric remove`'s report: survives.
  - Cancel on a gone pair: its reached-parent check reads the child as not found, so it survives.
- D4:
  - `DetachedPush` versus the exclusive teardown lock: survives, since `Remove` holds it.
  - Pause gate on the push: breaks (finding 4).
  - The coalesced push carrying the code branch: LOW (finding 5).
  - `respawnPairPush` after a failed `Remove`: survives.
- D5: rendered tackler stencil: survives.
- D6:
  - `removeWeftWorktree`, the `removeSibling` origin-only path and the cleanup sweep: lease fine.
  - Behind-origin remover followed by a later `Add`: breaks (finding 3).
- D7:
  - The callers of `DiscardTaskBranch` (cancel only, after `RemovePair` has released the lock): survives, no self-deadlock.
  - `rollbackAdd`: takes no teardown lock, so it survives.
  - `lockPairTeardown`'s only taker is `remove.go`: survives.
  - A `ParentLocked` push child waiting under a pause makes `Add` wait for `teardownWait` and then refuse. That is stated in the record, so it survives.

**Tool calls**

- Read Worktree-Create's `Add` body in `internal/battencli/wire.go` and listed every `Add` caller: my context had only the subagent's summary of that row and no list of callers.
- Grepped the callers of `DiscardTaskBranch`, `Add`'s refusals and `rollbackAdd`, and the pause-gate sites in gitrepo and gitexec: the existing refusals and the gate's location were not in my context.
- Read `WriteFailedStop`'s doc and request struct in `internal/shedengine/failedstop.go`, plus the `HoldRemoteRefs` grep: my context had only the function's name, not its lock and commit semantics.
