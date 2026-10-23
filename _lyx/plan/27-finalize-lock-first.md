# Card 27 — finalize-lock-first

**Create:**
- `internal/landingshed#Finalize.syncParent`
- `internal/landingshed#recordingParentMerger.FastForwardPair`
- `internal/landingshed#recordingParentMerger.CodeDirtyPaths`
- `internal/landingshed#TestFinalize_LockFirstAndFastForward`

**Edit:**
- `internal/landingshed#parentMerger`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#TestGuardedRows_LockLifecycle`
- `internal/landingshed#TestGuardedRows_MovedParentReroutes`
- `internal/landingshed#TestFinalize_StuckReasons`

**Uses:**
- `internal/landingshed#parentPuller`

**Extends:**
- `internal/landingshed/doc.go`

**Intent:** Finalize's block runs under the parent lock, taken first (D10 step 1), and the lock is released on every way out of the block.
`Call` takes the lock through the shared guard before anything that reads or moves the parent, then runs the run-scope origin reconcile and the owner checks as today, then the new `syncParent`.
`syncParent` is Merge-In's former parent sync moved here: it reads the parent's tips (fetching unless the push is skipped) and fast-forwards a local parent behind its upstream in its own pair, which moves it to the upstream tip and touches no local commit.
On every path, behind, equal, with no upstream and under the push skip alike, it first opens the parent pair and probes it: a pair that is missing, holds tracked dirt in its code checkout or is mid-merge is Stuck with the way forward Merge-In gave, before any merge, so the parent-side merge's guard is never the first to find them.
A parent behind its upstream is then fast-forwarded, and one no longer a fast-forward of its upstream is Stuck the same way; with no push skip, a local parent ahead of or diverged from its upstream is Stuck the same way; under the push skip it neither fetches nor fast-forwards.
`parentMerger` gains `FastForwardPair` and `CodeDirtyPaths`, which `*fabricengine.Fabric` satisfies, beside its `MidMerge`, and the test fake implements both; `parentPuller` stays for the compile-time assertion only.
Finalize no longer tests whether the parent moved since Merge-In: it merges the parent itself under the lock in the merge-judgement card, so `holdParent`'s moved check no longer runs here.
The lock is released on Done, on a reroute, on every Stuck and on every error, through one deferred release in `Call`; a lock a dead Finalize left is taken over at the next acquire, since `internal/parentlock` grants a record whose owner process no longer lives.
`pushParent` no longer releases the lock itself.

`TestFinalize_LockFirstAndFastForward` pins that the lock is held before the reconcile; that a parent behind its upstream is fast-forwarded under the lock; and that each parent-pair refusal, missing, dirty and mid-merge, is Stuck with its way forward before any merge with the parent equal to its upstream, with no upstream and under the push skip, and a not-a-fast-forward refusal from the behind state.
`TestGuardedRows_LockLifecycle` alone asserts the release on Done, a reroute, a Stuck and an error; the stuck-reason test drops Finalize's moved-parent reroute.

`doc.go`'s Finalize section states the lock-first block and its release.

**ImpactSummary:** Finalize's entry order and lock lifetime change, and the parent fast-forward moves into it from Merge-In; its fakes gain one method.
