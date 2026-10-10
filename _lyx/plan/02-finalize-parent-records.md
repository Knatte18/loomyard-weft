# Card 2 — finalize-parent-records

**Create:**
- `internal/landingshed#Finalize.commitParentRecords`
- `internal/loomcli#parentRecordsCommitter`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed/doc.go`
- `internal/landingshed/finalize_test.go`
- `internal/loomcli#landingDeps`
- `internal/loomcli/landingdeps_integration_test.go`
- `contracts/specs/refusal-spec.md`

**Intent:** D9: Finalize commits the parent pair's own run records before its parent-side merge, so batten's uncommitted status records of sibling runs no longer trip the merge guard (#501).
`Deps` gains `CommitParentRecords func() error`, told and nil-is-absent like `CommitStatus`: it commits the parent pair's run records, and a clean subtree is a no-op.
`Finalize.Call` calls `commitParentRecords` after the catch-up merge-in step and before `parentOpener`, on the first pass and on the merge-in-required retry alike;
a commit failure is Stuck with the reason `the parent pair's run records could not be committed: <err>; way forward: run "lyx fabric commit" in the parent pair, then "lyx loom resume"`, and every text of the step names the parent pair and its run records, never a side.
The merge guards are untouched.
`landingDeps` fills the seam with `parentRecordsCommitter`, whose closure resolves the parent pair at call time through `hubgeom.ResolveParent` and `lyxcwd.ResolveWorktree` over `fabricengine.WorktreePath`, and commits `shedrun.RunsRootRel()` on that location through `fabricengine.CommitAnchoredPaths` with a message naming the landing slug;
when the parent's run-records directory holds no file the closure returns nil without a commit, since git refuses a pathspec that matches nothing.
Tests: `finalize_test.go` gains a row asserting the Stuck reason and a step asserting the seam runs after the merge-in and before the parent merge, and that a nil seam is skipped;
the loomcli integration test over a real hub gains a step dirtying the parent's run records and asserting the closure commits them and leaves a dirty path elsewhere alone.
The refusal spec's loom section gains the row `Finalize: parent run records commit failed` with that way forward.

**ImpactSummary:** One new told seam on `landingshed.Deps`, filled by loomcli; Finalize gains one step between its merge-in and its parent merge; no other producer changes.
