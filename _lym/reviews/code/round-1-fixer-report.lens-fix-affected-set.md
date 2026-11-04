```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix covers every member of the sets under F1, F4 and F5 (parent-side merge sites, `atOnce` sites, `pushParent` callers).

**Findings**

- LOW, `internal/landingshed/doc.go:258`. This is a stale sibling of the F4 change. The package doc says a yes whose pull request GitHub reports closed with no operator's ending is Stuck, and its only exception is a yes marked `MergeAtOnce` with no record. After the fix, `finalize_pr.go:206` also reopens a closed recorded pull request under `MergeAtOnce`. The doc's line 258 and the nearby line 262 give no hint of that.
- LOW, `internal/landingshed/doc.go:263`. The doc says a `MergeAtOnce` yes always merges by squash. After F1 that holds for the parent-side merge too, but the doc never says so.
- LOW, `internal/landingshed/doc.go` (protected-branch way forward). I did not search for text about the protected-branch way forward, so I could not confirm that doc.go's description of it matches the new `parentMerges` variant. Treat it as unchecked.

**Checked, no gap**

- F1: `finalize.go:337` is the only `MergeOptions{` site outside tests in `landingshed` and `mergeresolve`. The retry merge at `finalize.go:360` reuses the same `mergeOpts`, so both parent-side merges now squash.
- F5: both `pushParent` callers (`finalize.go:342` and `finalize.go:365`) pass `landing.atOnce`. `protectedParentWayForward` has one non-test caller, and the two test call sites in `finalize_test.go` were updated.
- F2 and F4: every `atOnce` site outside tests (`finalize_pr.go:196`, `204`, `212`, `273`, `349`) is consistent with the new condition at `finalize_pr.go:206`.

**Not checked:** F3 (the record assertions) and F6 (the comment) have no set premise.

**Tool calls**
1. A grep for `Config.Squash` and `MergeOptions{` over `landingshed` and `mergeresolve` (F1's set). My context did not hold the sites in other files or packages.
2. A grep for `pushParent(`, `protectedParentWayForward(`, `MergeAtOnce` and `atOnce` over non-test Go files (the F4, F5 and F2 sets). It also showed which doc comments name `MergeAtOnce`.
