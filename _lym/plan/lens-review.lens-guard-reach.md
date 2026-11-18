```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal and test row the cards add, I traced whether a real entry point can reach it and whether the guarded site recurs unguarded.

**BLOCKING** — Cards 4, 5 and 14, the protected-refusal rows. The planned hook output never reaches `ProtectedBranchRefusal`, so all three rows fail.
- The rows are:
  - card 4's `AProtectedBranchRefusalIsRecordedProtectedAndIsNotOwed`;
  - card 5's `ProtectedMarkerPushLeavesNothing`;
  - card 14's `ProtectedRefusalRecordsProtected`.
- Each plants a pre-receive hook that prints `remote: error: GH006: Protected branch update failed…` and exits 1.
- `internal/gitrepo/push.go:251-268` (`mapPlainPushError`) wraps `ErrPushRejected` with only the last stderr line holding a `rebaseRetryTriggers` word (`push.go:49`: "non-fast-forward", "rejected", "fetch first").
- For a local hook that line is `! [remote rejected] <b> -> <b> (pre-receive hook declined)`. It holds neither "protected branch" nor "repository rule violations", so the class comes out as `other` or `not_fast_forward`, never `protected`.
- GitHub's own rejection line, `(protected branch hook declined)`, would match, but no row reproduces it.
- The fix is either to match the full stderr, or to write the hook so that the rejection line itself carries the marker.

**BLOCKING** — Card 10, the claim that "every records commit reaches" `commitRecordsPathsHeld`, is false. Two commit sites bypass the mid-merge route, so they still fail during a merge.
- `CommitEmptyWithTrailers` commits through `CommitPathAllowEmpty` directly (`internal/fabricengine/recordsgit.go:227`). Its caller is `internal/loomcli/decision.go:1562`, which runs `lym loom decision add`.
- `Fabric.commitWeftLocked` commits through `f.records.StageAndCommit` (`recordsgit.go:268`). This is the path `lym fabric sync`, `lym fabric commit` and `webstercli#fabricSync` take, through `Fabric.commitBothSides`.
- In a one-repository worktree with `MERGE_HEAD` present, both still hit git's "cannot do a partial commit during a merge".
- That also makes card 10's "`Fabric.Commit` takes the route for records-only file lists" unreachable as written. The route needs to sit in those two primitives too.

**MEDIUM** — Card 10's records-only route in `Fabric.Commit` is unreachable over a fabric merge record.
- `Fabric.Commit` returns `*ErrMergeInProgress` on `mergeRecordExists` before the foreign-state check (`internal/fabricengine/commit.go:104-110`).
- The card only replaces the foreign-state refusal.
- So `lym fabric sync` or a webster sync over a fabric merge stopped before step 3, or parked after it, still refuses. The unsettled-merge commit-and-abort route is then reachable only through `CommitRecordsPaths`.

**MEDIUM** — Card 10's row `CrashBeforeStepThreeCommitsAbortsAndPushes` cannot produce its state with the hook card 9 declares.
- The row needs "a merge stopped by the card 9 hook returning a crash".
- Card 9 declares `SetAfterMergeStartForTest(hook func())`. A `func()` hook returns nothing, so it cannot make `Fabric.Merge` exit between steps 2 and 3 with `MERGE_HEAD` and an unsettled record left behind.
- The fix is to have the hook return an error that `Merge` returns as-is, with no self-abort.

**MEDIUM** — Card 12: teardown's duplicate sites recur unguarded once the records and code sides are one branch and one worktree.
- With card 3's geometry, `RecordsWorktreePath(l, slug)` equals `WorktreePath(l, slug)` and `recordsBranchIn` equals the task branch.
- `Topology.removePair` removes the weft worktree and branch and then runs `deleteWarpBranch` (`remove.go:622-655`) on the same branch.
- `runRemoteTeardown` runs `deleteWeftBranchOnRemote` (`cleanup.go:417-458`) and then `deleteTaskBranchOnOrigin` (`remove.go:657-680`) on the same origin branch. The second leased delete meets a gone branch or a moved lease.
- Card 12 keeps "today's order" and only adds the view ref. It never names the second removal and the second delete as skipped, so `TestRemove_OneRepository` would hit them.

**MEDIUM** — Card 9's marker-change refusal is reachable by a fabric-made change: Add `--anchor` in a one-repository hub.
- `refuseSourceMarkerChange` refuses any source commit that changes `.lym-anchor` without `AnchorChangeTrailerKey`.
- Add `--anchor` writes a new marker through `moveForkedWeftToAnchor` and then `stageNewMarker` (`add.go:642`, `addanchor.go:88`). That lands in Add's first records commit, which card 6 builds with no trailer.
- So every merge of a beam added at another anchor refuses. Card 6 never adds the trailer, and card 8 only adds it to `moveWeftContent`.

**MEDIUM** — Cards 9 and 13: the "bring a branch level with its own origin" site in `Fabric.Merge` stays unguarded.
- `Fabric.Merge` calls `syncSideBeforeMerge` (`merge.go:445`), which runs `MergeFFOnly` and refuses a diverged side as "branch not synced to upstream".
- D5 says that bringing a branch level with its origin tip commits the records, replays the local records-only commits, and refuses only on an unpushed non-records commit.
- A receiving parent with an unpushed records-only commit (an owed or guarded prime push, for example) whose origin moved is refused at this site. Card 13's replay is wired only into Pull, `retryWeftPush`, reconcile and `FastForwardPair`.

**BLOCKING** — Card 14's prime guard recurs unguarded on the detached push path.
- The guard sits only in `PushPairAnchored`.
- A prime records commit made through `Fabric.Commit` (`lym fabric sync`, a webster sync in the prime) spawns `SpawnDetachedPush` (`commit.go:166`). That path is `DetachedPush`, then `CoalescePushRequest`, then `coalescePushSides` (`coalesce.go:118-203`), then `pushSideDetached`, then `pushClassified`.
- It pushes the prime's branch, now the operator's code branch, with no `primePushOperatorCommits` check. `retryWeftPush` on a rejected prime push pushes it too after a replay.
- Card 12 edits `coalescePushSides` but adds no guard there.

**LOW** — Card 7: `refuseOffPrimeBranch` recurs unguarded on the merge path.
- It is called only from `CommitRecordsPaths`, `CommitEmptyWithTrailers` and `commitWeftLocked`.
- Card 9's step 1 and card 10's `MergeAbort` commit records through `commitRecordsPathsHeld` directly.
- A local landing or merge-in whose receiving worktree is a prime switched off `lym.primeBranch` therefore commits the prime's records onto the wrong branch.

**LOW** — Card 7: the `index.lock` retry does not cover card 10's commit-around-merge.
- `retryHeldIndexLock` wraps three commit calls.
- `commitRecordsAroundMerge` (card 10) runs `update-index` on the worktree's index and `update-ref` with no wrapping.
- A held `index.lock` there fails at once, while D4 says a records commit meeting a held `index.lock` retries.

Tool calls beyond my context:
- I grepped `internal/gitrepo/push.go` to see what text a push rejection error carries; my context had only a summary of the triggers.
- I read `internal/gitrepo/push.go:44-68` for `rebaseRetryTriggers`, the list that selects the line kept in that error.
- I grepped the non-test callers of `CommitEmptyWithTrailers`, `syncSideBeforeMerge`, `stageNewMarker` and `moveForkedWeftToAnchor` to confirm which real entry points reach those sites.
