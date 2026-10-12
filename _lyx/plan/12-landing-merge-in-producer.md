# Card 12 — landing-merge-in-producer

**Create:**
- `plan:internal/landingshed#mergeInName` -> `const mergeInName = "Merge-In"`
- `plan:internal/landingshed#parentPuller` -> `type parentPuller interface`
- `plan:internal/landingshed#MergeIn` -> `type MergeIn struct`
- `plan:internal/landingshed#NewMergeIn` -> `func NewMergeIn(deps Deps) (*MergeIn, error)`
- `plan:internal/landingshed#MergeIn.Call` -> `func (m *MergeIn) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/landingshed#MergeIn.syncParent` -> `func (m *MergeIn) syncParent(ctx context.Context) (string, string, error)`
- `plan:internal/landingshed#MergeIn.writeRecord` -> `func (m *MergeIn) writeRecord(tip, head string, merge mergeresolve.Result) error`
- `internal/landingshed/mergein_test.go`

**Edit:**
- `internal/landingshed/doc.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/landingshed#newParentGuard`
- `plan:internal/landingshed#parentGuard.ensure`
- `plan:internal/landingshed#computeOverlap`
- `plan:internal/landingshed#WriteMergeRecord`
- `plan:internal/landingshed#ReadReviewedTip`
- `plan:internal/fabricengine#Fabric.FastForwardPair`
- `internal/landingshed#preMerge.clear`
- `internal/landingshed#verifyGate.check`

**Intent:**
Add the `Merge-In` producer (D1) as a constructed but not yet listed producer; card 17 puts it into the recipes.
Its steps, in order:

1. `CommitStatus` as Publish does today.
2. `parentGuard.ensure`: Merge-In takes the parent lock before it touches the parent.
3. `syncParent` brings the local parent branch level with its upstream and returns the tip to merge, or a Stuck reason.
   - Under the told push skip it neither fetches nor fast-forwards, and returns the local tip.
   - Otherwise it reads `ReadParentTips(true)`.
   - `ParentNoUpstream` and `ParentEqual` return the local tip.
   - `ParentAhead` and `ParentDiverged` are Stuck before any pull runs: `the local parent branch "<branch>" is ahead of its upstream` (or `has diverged from its upstream`) `; way forward: push or reconcile "<branch>" in its own pair, then run "lyx loom resume"`.
     Merge-In never pushes or rewrites the parent.
   - `ParentBehind` opens the parent pair through `OpenParentFabric` and calls `FastForwardPair` through the narrow `parentPuller` seam, mirroring Finalize's `parentMerger`.
   - An open failure is Stuck: `no live pair for parent branch "<branch>": <cause>; way forward: re-create the parent pair, then run "lyx loom resume"`.
   - `FastForwardDirty` is Stuck: `the parent pair of "<branch>" has uncommitted changes; way forward: commit or stash them in the parent pair, then run "lyx loom resume"`.
   - `FastForwardMergeInProgress` is Stuck: `a merge is in progress in the parent pair of "<branch>"; way forward: finish or abort that merge in the parent pair, then run "lyx loom resume"`.
   - After a fast-forward it returns the upstream tip.
   Bound: the fast-forward runs only under the lock, only from the behind state, and only moves the parent pair to its upstream tip, so it never touches local commits and cannot race another run's Finalize, which merges into the same worktree under the same lock.
4. It reads the task head before the merge (`TaskHead`), the parent tip last merged into the task (`MergeBase(parentBranch)`) and the parent tip the last finished merge review reached.
   That last tip is `ReadReviewedTip(MergeReviewedPath)`, else the earlier merge record's `reviewed_tip`, else that merge base, the fork point.
5. The pre-merge clear, the clean check, the resolver's merge-in of the parent branch, the clean check, the plan verify through the verify gate, and the last clean check.
   These are Publish's steps today, with the same Stuck reasons.
   The gate carries a failure recorder, so a failed plan verify writes the failure record with row `Merge-In` and the told target, and a passing one clears it.
6. `writeRecord` writes the merge record: `merged_tip` is the tip `syncParent` returned; `task_base`, `task_head` and `reviewed_tip` come from step 4; `merge_commit` is HEAD after a merge that was not already up to date; `up_to_date` is the resolver's answer.
   The overlap is `computeOverlap(ChangedPaths(reviewed_tip, merged_tip), ChangedPaths(task_base, task_head))`; an up-to-date merge records an empty overlap.
   A record write failure is returned as an error, never a Stuck.
7. Done.

Every Stuck goes through the usual cancellation check, and the lock's paused and unreadable reasons are returned with `Halt`.
`NewMergeIn` refuses a nil `OpenFabric`, `OpenParentFabric`, `ReadParentTips`, `ChangedPaths`, `MergeBase` and `TaskHead`, a nil `Lock.Store` and an empty `MergeRecordPath`, each with its own message, and builds the resolver as `NewPublish` does.

`mergein_test.go` drives `Call` with a fake resolver, gate, tip reads and parent puller, and a real `parentlock.Store` in a temporary directory:
- `CommitStatus` runs before the lock and the merge;
- a lock held by another run makes the call wait, then proceed;
- each `syncParent` state: equal, no upstream, the push skip with a moved upstream ignored, behind with the fast-forward recorded, ahead, diverged, no parent pair, dirty parent pair and merge in progress; the last five are Stuck with their way forward and no merge run;
- the merge record's fields for a fresh merge, an up-to-date merge and a second merge whose `task_base` is the first merge's tip, so earlier merged parent changes never count as the task's;
- a failed plan verify writes the failure record with row `Merge-In`;
- an undecodable lock record is a halting Stuck naming `lyx batten lock release`;
- `NewMergeIn`'s refusals.

`refusal-spec.md`'s loom section gains a row for each of Merge-In's five parent-sync Stucks and for the lock's paused and unreadable halts, and the doc gains a Merge-In paragraph.

**ImpactSummary:** a new landing producer no recipe lists yet; nothing existing changes behaviour.

**Commit:** 12: landing: the Merge-In producer
