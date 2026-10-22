---
verdict: APPROVED
---

No defect found in the range `4842d72fc^..HEAD` (cards 1–6), judged against the plan and the rubric.

Focus questions settled:

- Cards 3, 5 and 6 all word the dirty-parent refusal through `landingshed.DirtyParentRefusal`. The Finalize reason's dirt-only arm calls it, and `finalizeDirtGuard` calls it for both the goto guard and the driver start check. The start check is wired with `c.finalizeGuard()`. No second rendering of the checkout, paths or way forward exists.
- Card 3's merge-in-progress way forward saves a copy outside the checkout, as the overview's stated departure from D2 says. The departure is from the decision record, not from a PATTERN entry, and no entry forbids the wording, so a `Pattern-Departure:` trailer is not owed. No declared PATTERN change is owed either.
- Card 4 removes `ParentStatus`, `ParentFabricStatus`, `renderParentFabricStatus`, `parentDirtyPathsFromFabric`, the `parentDirtyPaths` field and `IsWorktreeDirtyReason`. A repo-wide grep finds no remaining reference except the tolerant `parentHeader` skip in `Parse` and the tests asserting its absence.
- Card 1's guard error carries `Checkout` and `DirtyPaths` outside `Error()`, and `pairDirtyReason` decides on the same list it returns, over the same `--untracked-files=no` scope as before.
- `Fabric.CodeDirtyPaths` and `Fabric.MidMerge` are read-only. The goto guard and the start check read the code checkout alone and admit when a merge record exists or the read fails.
- Test economy: every added behaviour has a pinning row, and no duplicate or per-helper test was found.

## Rejected

- `TestIsFinalizeRow` lacks `t.Parallel()` (lens:code-pattern-fit, lens:code-tests): no test in `bouncerrows_test.go` calls `t.Parallel()`, so the file's convention is serial and the new test follows it.
- A nil `parentCodeDirt` could panic in a hand-built `loomCLI` (lens:code-correctness observation): `newLoomCLI` always sets it, and no test in the repo goes through `settledGuard()` or `finalizeGuard()` to Finalize with an unset seam.
- Unasserted `withDirt` sites in `Merge` and `MergeIn` (lens:code-tests): `merge_target_integration_test.go` and `mergein_recovery_integration_test.go` assert the fields through `assertGuardDirt`.
