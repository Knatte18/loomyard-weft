```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 change set for sibling or twin inconsistencies (lens fix-consistency).

no findings

- F1: `conflictingFeature("clash.txt")` runs the same two steps the old test did, in the same order. The `wantStartSHA` and `wantSourceSHA` reads still follow the fixture call. The other `clashLayer` sites in `mergein_recovery_integration_test.go`, `mergelock_integration_test.go` and `mergeweftlocal_integration_test.go` use the same call form.
- F2: the other "Arrange" mentions in `livestate_verbs_test.go` concern verbs whose steps stay in `Arrange`. The Pull comments at lines 109, 112, 903, 942 and 963 are examples. Line 606 is also fine: it says Arrange reads `OriginalBranch`, and Cleanup's `Arrange` still does that. No twin of the Cleanup comment was left stale.

Tool calls:
1. One `grep` over `livestate_verbs_test.go` for "Arrange" and over the test files for `conflictingFeature`. My context lacked the other "Arrange" mentions and the `conflictingFeature` body, which I needed to check F1's equivalence and F2's twins.
