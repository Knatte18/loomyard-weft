```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set. I reran the F4 and F2 set searches. F1, F3, F5, F6 and F7 are single-site or table-row fixes, so I left them unchecked.

**LOW — F4, `internal/testcover/testselect.go:218`.** The fix relaxes the selector's `BuildFailure` check only.
- The two other readers of `Map.BuildFailure` still treat any non-empty value as a failed map, whatever tests it holds.
- `internal/gatecli/redundancy.go:305` (`case m.BuildFailure != "":`) reports "its coverage map records a build failure".
- `internal/gatecli/testmap.go:459` (`m.BuildFailure != "" && m.Commit != head`) rebuilds the map.
- Today `writeFailed` (`internal/gatecli/testmappackage.go:371-372`) sets `Tests = nil` whenever it sets `BuildFailure`, so no real map holds both and nothing breaks.
- A map that did hold both would be usable to the selector but reported failed by redundancy and rebuilt by testmap. That follows from reading D5's field as `Map.BuildFailure`.

**F2:** no findings. `compareTestSources` has one caller, `compareTestFile` (`testselect.go:479`), which has one caller in `packageChanges` (`testselect.go:294`). That call site now judges both `addedImports` and `removedImports`. `maps` is still used in `declcompare.go`, by `sortedKeyUnion`.

Tool calls:
1. grep for `BuildFailure` and `MissBuildFailure` under `internal`, to find every site that reads the failure record. My context held only the files the findings named.
2. grep for `compareTestSources`, `compareTestFile`, `testFileComparison` and `maps.Equal` in `internal/testcover`, to find every caller of the changed comparison. My context did not show them all.
