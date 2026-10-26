```yaml
lens: fix-affected-set
outcome: reported
```

Findings checked: F1 (the only one whose premise is a set of sites). The F2, F3 and F4 premises are single tests or rows, not sets.

No findings.

- F1: no unit comparison of the same kind is left in `internal/planparser/*.go` outside tests. The two remaining `.Unit ==` and `.Unit !=` sites, `internal/planparser/targetdirs.go:294` and `:296`, are in a different function. The first already handles the external-test-package case through `goPackageDir`. The second is the non-Go branch (the context shows the `plan.GoToolchain()` guard only in the first line), so the external-package gap does not reach it.

Left unchecked: none.

Tool calls:
1. `grep -n "\.Unit != \|\.Unit == " internal/planparser/*.go | grep -v _test.go`. This reran the F1 search for sibling unit comparisons, which my context did not show.
