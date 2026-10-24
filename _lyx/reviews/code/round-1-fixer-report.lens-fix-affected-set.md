```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

I checked F3, the only finding whose premise is a set. `scope.go` has no `sort.` call or `"sort"` import left, so the file no longer uses the package. Other `gatecli` files still use `sort.Ints` (`tiers.go`) and `sort.Strings` (`tierrun.go`, `green.go`, `tiers.go`). The review named only `scope.go:402`, and I treat those as out of scope.

Left unchecked: F1 and F2. They are coverage gaps in one test file, not sets under one key.

Tool calls:
1. `grep` for `"sort"` imports and `sort.*` calls across the non-test files of `internal/gatecli`, to rerun F3's search beyond `scope.go`. My context only held the `scope.go` grep.
