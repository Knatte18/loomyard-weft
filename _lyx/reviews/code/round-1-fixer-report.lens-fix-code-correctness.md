```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code the fix changed (F2, F3, F5 and F6 in `internal/gatecli`) for correctness, using the fix-code-correctness lens.

No findings.

- **`newestWholeAncestor` (`green.go`), history ordering:** I ran the entries `new(0)`, `old(1h)` with the chain old→new. `new` is taken first. `old` is accepted, then `isAncestor(new, old)` is false and `isAncestor(old, new)` is true, so `old` is skipped and the result is `new`. The old Written-only code would pick `old`, so the new row fails with the defect in place.
- **`newestWholeAncestor`, other orders:** The order `old(0)`, `new(1h)` replaces `old` with `new` through `isAncestor(newest, held)`. Commits unrelated by history (the `everyCommit` row) fall to the Written comparison, and `mid` does not displace `new`.
- **`newestWholeAncestor`, failures:** A failing `IsAncestor` call returns false, so the two commits count as unrelated and the older Written time decides. That is the safe direction and only widens the diff.
- **Caller of the changed signature:** `scope.go` was the only non-test caller, and its closure now takes `(commit, of)` and passes `of` through. The tier1 gate run compiled and passed.
- **`with` pruning (`green.go`):** An entry with args is no longer counted against `greenWholeCommitsKept` and survives only for the current tree. Other-tree entries are never read by `covered`, which requires `held.Tree == tree`, so dropping them loses nothing.
- **New pruning subtest:** After 20 filtered writes the record holds `commit-first` and `commit19`, in that order. With the old code the 21 whole entries would exceed 20 and evict `commit-first`, so the subtest fails with the defect in place.
- **`TestGreenRunRecordPass` (F2):** The case that moves HEAD gives a different tree, so `stillGreen` is false and no entry is written. Removing the guard records one entry, so the test fails with the defect in place. The unmoved case is a positive control.
- **F3 rows:** `-skip`, `-list`, `-bench`, `-test.run=` and `--run` all pass through `isFilterFlag`'s dash and `test.` trimming, and the gate run was green.

Tool calls: none. My context already held every changed file.
