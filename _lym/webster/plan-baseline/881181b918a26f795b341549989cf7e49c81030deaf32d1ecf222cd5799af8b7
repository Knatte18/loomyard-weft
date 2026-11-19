# Card 12 — one-branch-push-and-teardown

**Create:**
- `plan:internal/fabricengine#ViewRefPrefix` -> `const ViewRefPrefix = "lym-view/"`
- `plan:internal/fabricengine#deleteViewRefOnOrigin` -> `func deleteViewRefOnOrigin(rec *Mutations, repoDir, slug string) error`
- `plan:internal/fabricengine_test#TestPairPush_OneRepository` -> `func TestPairPush_OneRepository(t *testing.T)`
- `plan:internal/fabricengine_test#TestRemove_OneRepository` -> `func TestRemove_OneRepository(t *testing.T)`

**Edit:**
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#statusPushRequest`
- `internal/fabricengine#coalescePushSides`
- `internal/fabricengine#pairedCodeWorktree`
- `internal/fabricengine#pushOwedCodeTip`
- `internal/fabricengine#ReturnToPreviousTransition`
- `internal/fabricengine#Topology.removePair`
- `internal/fabricengine#remoteTeardownJob`
- `internal/fabricengine#runRemoteTeardown`
- `internal/fabricengine_test#TestRunTeardownChild_RunsTheRemoteStepsInOrderUnderTheLock`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#OriginCodeTip`
- `internal/fabricengine#HandOverTips`
- `internal/fabricengine#deleteRemoteBranch`
- `internal/fabricengine#archiveWeftTip`
- `internal/fabricengine#oneRepositoryAt`
- `internal/hubforge#NewOneRepoHub`
- `internal/hubforge#CloneSecond`
- `internal/hubforge#AddPairWith`

**Intent:** D6: a pair's push, hand-over and teardown in a one-repository hub, where one branch carries records and code.

- `statusPushRequest` sets no `WarpPath` for a one-repository pair, since its records worktree is its code worktree,
  and `coalescePushSides` skips the code side whenever its path equals the records side's, so `CoalescePushBothAt` with one worktree for both, as `AddPairWith` passes it, pushes once.
  `PushPairAnchored` and `coalescePushSides` then push the one branch once, as the records side, under one push lock and the slug's teardown lock;
  "records first, code only once it landed" holds trivially. The retry, the owed-push hold and the failed-push record with its creel notice stay,
  and the record names the branch once, as the records side, so `PushOwed` and the act rule read it as today. A rejected push reads the run record at origin's fetched tip and hands over on another machine's takeover, as today.
- `pairedCodeWorktree` returns the worktree itself in a one-repository hub, so every records-side failed-push record carries it as `CodeWorktree`:
  its way forward names `lym fabric push` in the worktree's anchored directory, and its notice carries the pair's slug as its run, which the pair's teardown drops.
- No code tip can be owed apart from its records there: `pushOwedCodeTip` returns not pushed without acting (so `deleteGivenUpCodeBranch` never runs), and `ReturnToPreviousTransition` returns false committing nothing.
  `OriginCodeTip` needs no change: with card 3's branch naming it reads the trailer's commit, an ancestor of the pushed tip, which origin holds.
- Teardown (`Topology.removePair`, `remoteTeardownJob`, `runRemoteTeardown`): the local step tags the branch tip `archive/<slug>/<tip>` before deleting the branch locally, in today's order and locks,
  and removes the one worktree and deletes the one branch once: the warp worktree removal and `deleteWarpBranch` are skipped, since they would name the same worktree and branch again.
  The detached child pushes the tag, then deletes the branch on origin leased on its tip through the records-branch delete alone (`DeleteTaskOnOrigin` false, so `deleteTaskBranchOnOrigin` does not run),
  then deletes `refs/heads/lym-view/<slug>` (`ViewRefPrefix`) on origin through `deleteViewRefOnOrigin`.
  `deleteViewRefOnOrigin` lives in `destroy.go`, the destruction chokepoint, and reads a ref already gone on origin as success.
  The job's new field `DeleteViewRef` is set only for a one-repository pair; a zero value deletes no view ref, so a job an older binary wrote reads as today.

Tests:
- `TestPairPush_OneRepository` (tier1, `NewOneRepoHub`, a pair through `AddPairWith`) rows:
  `PushesTheBranchOnce` (one transition commit; after `PushPairAnchored` origin's branch is the local tip and the result records one branch push);
  `RefusedPushLeavesOneOwedRecord` (origin unreachable: one failed-push record of the records side naming the run, read owed by `FailedPushesOwedBy`, its `CodeWorktree` the pair and its way forward naming `lym fabric push` in the pair's anchored directory);
  `TakeoverKeepsEveryTransitionOnOrigin` (a `CloneSecond` hub writes a takeover entry and pushes; the first hub's next push hands over and origin's branch still holds every transition commit);
  `NoCodeTipIsOwed` (two transitions committed and pushed, then a third committed locally: `OriginCodeTip` reads not missing, `ReturnToPreviousTransition` returns false committing nothing, and `HandOverTips` pushes no code tip and deletes no origin branch).
- `TestRemove_OneRepository` (tier1) rows: `TagIsPushedTheBranchAndTheViewRefGo` (a `refs/heads/lym-view/<slug>` planted on origin first; after the removal origin holds the archive tag at the tip, no branch and no view ref, and no local worktree or branch is left);
  `ViewRefAlreadyGoneIsNoFailure`.
- `TestRunTeardownChild_RunsTheRemoteStepsInOrderUnderTheLock` (tier1) gains the row `OneRepositoryTagThenBranchThenViewRef`: a job with `DeleteViewRef` set records the tag push, then the branch delete, then the view-ref delete, in that order.

**ImpactSummary:** Pair push, failed-push records, code-tip settling and teardown gain one-repository branches; the teardown job record gains a field an older job reads as zero; two-repository pushes and teardowns are unchanged.
