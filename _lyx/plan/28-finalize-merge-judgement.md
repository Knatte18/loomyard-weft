# Card 28 — finalize-merge-judgement

**Create:**
- `plan:internal/landingshed#taskMerger` -> `type taskMerger interface`
- `plan:internal/landingshed#Finalize.mergeParent` -> `func (fz *Finalize) mergeParent(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `plan:internal/landingshed#Finalize.rerouteReason` -> `func (fz *Finalize) rerouteReason(cause string) string`
- `plan:internal/landingshed#TestFinalize_MergeJudgement` -> `func TestFinalize_MergeJudgement(t *testing.T)`
- `plan:internal/landingshed#fakeTaskMerger` -> `type fakeTaskMerger struct`

**Delete:**
- `internal/landingshed#Finalize.mergeInStep`

**Edit:**
- `internal/landingshed#Finalize`
- `internal/landingshed#NewFinalize`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Deps`
- `internal/landingshed#TestFinalize_MergeInRequiredRetry`
- `internal/landingshed#TestFinalizeVerify_RetryMergeInIsBracketedAgain`
- `internal/landingshed#TestFinalizeVerify_Lands`
- `internal/landingshed#newFinalizeVerifyFixture`
- `internal/landingshed#TestPreMerge_ClearsOwnParkedMergeIn`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#LandingDeps`

**Uses:**
- `internal/landingshed#computeOverlap`
- `internal/landingshed#ReadMergeRecord`
- `internal/landingshed#WriteMergeRecord`

**Extends:**
- `internal/landingshed/doc.go`

**Intent:** Finalize merges the parent in under its lock and judges the merge (D10 steps 2 and 3), replacing its catch-up merge-in.
`mergeParent` reads the merge record's merged tip, clears a parked merge-in and checks the tree clean as today, then merges the parent branch into the task worktree.
A loom Finalize (the new told `Deps.ConflictReroutes`, true for loom and false for darn, filled by `landingDeps` from the recipe) merges through `taskMerger`, the task pair's `MergeIn` and `MergeAbort`, which `*fabricengine.Fabric` satisfies: on a conflict it aborts the merge and reroutes, leaving the merge and its conflict session to Merge-In outside the lock.
A darn Finalize merges through the resolver, which resolves a conflict with the conflict session under the lock, since darn has no other merging row.
`ConflictReroutes`' zero value, false, takes the darn path, so a struct-literal Finalize merges through its resolver as today.
An unreadable merge record is logged and read as absent, so the parent side is measured from the task's merge base, as a darn run with no record is; Finalize rewrites the record afterwards.
It then judges the merge large when it needed the conflict session, or when the parent's changes since the merged tip it read (`ChangedPaths(merged tip, new tip)`) touch a file the task changed (`ChangedPaths(task merge base, task head before the merge)`), through `computeOverlap`'s file set as the retired skip seam took it; a darn run with no merge record measures the parent side from the task's merge base.
It writes the merge record with the new merged tip, keeping the aligned tip as it was.
A large merge keeps the merge commit and reroutes, with no gate run and no fixer.
A reroute is a Stuck with Reroute on the row's `on_reroute` edge, never a goto, and spends no bounce budget; the deferred release frees the lock first, and shed's reroute cap bounds it.
`rerouteReason` words each reroute, naming its cause: a conflict, a large merge with the overlapping files, or, in the gate card, a spent fixer cap or an over-cap fix.
After a merge-in-required error from the parent-side merge, Finalize re-runs steps 2 to 4 once under the same lock, as its retry does today.

`fakeTaskMerger` is the test's `taskMerger`: its `MergeIn` answers a scripted result, conflicts included, and it records whether `MergeAbort` ran.
`TestFinalize_MergeJudgement` pins: an up-to-date merge proceeds to the gate; a loom conflict aborts the merge, recorded on the fake, and reroutes with no gate run; an unreadable merge record measures from the task's merge base; a darn conflict runs the conflict session and reroutes as a large merge; an overlapping parent change reroutes with the merge commit kept and no gate run; a non-overlapping parent change proceeds; a darn run with no merge record measures from the task's merge base; and the merge record carries the new merged tip with the aligned tip unchanged.
The retry, verify-fixture and pre-merge tests follow the new step.

`doc.go`'s Finalize section describes the merge and its judgement.

**ImpactSummary:** Finalize's catch-up merge-in becomes the judged parent merge, with one told seam added and filled by the landing assembly and the test kit.
