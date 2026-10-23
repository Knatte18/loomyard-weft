# Card 25 — merge-in-unlocked

**Create:**
- `internal/landingshed#MergeIn.mergeTip`
- `internal/landingshed#TestMergeIn_MergeTip`

**Delete:**
- `internal/landingshed#MergeIn.syncParent`
- `internal/landingshed#TestMergeIn_SyncParent`

**Edit:**
- `internal/landingshed#MergeIn`
- `internal/landingshed#NewMergeIn`
- `internal/landingshed#MergeIn.Call`
- `internal/landingshed#newMergeInFixture`
- `internal/landingshed#TestMergeIn_RunsStatusCommitThenLockThenSyncThenMergeAndVerify`
- `internal/landingshed#TestMergeIn_UnreadableLockRecordHaltsNamingTheReleaseVerb`
- `internal/landingshed#guardedRows`
- `internal/landingshed#TestGuardedRows_LockLifecycle`
- `internal/landingshed_test#TestMergeIn_ConflictSessionLoadsLanguageSkills`

**Extends:**
- `internal/landingshed/doc.go`

**Intent:** Merge-In takes no parent lock and moves no parent (D2).
It keeps its status commit, its run-scope origin reconcile and its clean-tree checks, and writes nothing in the parent pair: the fast-forward of the local parent branch and the parent-pair Stucks (ahead, diverged, missing, dirty, mid-merge, not a fast-forward) leave Merge-In, for Finalize to take under its lock.
`MergeIn` drops its `parentOpener` field, `NewMergeIn` stops building it, and `syncParent` goes.

The new `mergeTip` picks the parent tip to merge.
With an upstream and no push skip it fetches and takes the parent's upstream tip from its own fetching read of the parent's tips (`ReadParentTips(true)`), never the local tip, so a landing another run made locally but has not pushed is never merged in; a failure of that read stays the error stop it is today.
With the push skipped, or with no upstream, it takes the parent lock through the shared guard only for the read of the local tip and releases it before returning, so the merge itself runs unlocked; a paused lock wait and an unreadable lock record stay Stuck with Halt, as today.
`Call` then merges that tip through `resolver.Resolve(ctx, parent branch, tip)`, which reaches card 1's `MergeInAt`; a missing upstream ref and a mid-merge task pair stay Stuck with their way forward as the merge path words them today.
The rest of `Call` is unchanged for now: the `landing` gate through `verifytree.Verify` and the merge record with the tip it merged.
Merge-In no longer holds the lock on any way out, so no later row inherits it.

`TestMergeIn_MergeTip` pins, over the fixture's fake tips and lock store: with an upstream and no push skip, the upstream tip is merged while another run holds the parent lock and no lock is taken; a local parent ahead of its upstream merges the upstream tip, so the local-only commit is not merged, and the resolver saw that SHA; with the push skipped, and again with no upstream, the lock is held during the local-tip read and released before the resolver runs; a resolver answer that the upstream ref is missing is Stuck with its way forward; a failed tips read is the returned error.
The ordering, lock lifecycle and unreadable-lock tests drop the lock and sync steps from Merge-In and keep them for the push-skipped read.
`TestMergeIn_ConflictSessionLoadsLanguageSkills`, the real-hub test, gains a row where the local parent branch holds a commit its upstream lacks: Merge-In, through the real `mergeresolve.Resolver` and `Fabric.MergeInAt`, merges the upstream tip's SHA, the task tree lacks the local-only commit's file and the local parent branch's ref is unchanged, which pins the criterion at Merge-In through the resolver's merge seam.

`doc.go`'s Merge-In section states the unlocked merge and the tip choice.

**ImpactSummary:** Merge-In's lock and fast-forward go and its tip choice changes; its fixture and lock tests follow, and Finalize still runs its own catch-up merge until its card.
