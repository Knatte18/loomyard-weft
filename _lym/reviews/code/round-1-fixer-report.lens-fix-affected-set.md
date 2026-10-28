```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The `2*unchanged > total` comparison at `internal/commentlint/lint.go:200` is the only majority comparison in the package; its only caller is `lint.go:295`, driven by the one test table. Unchecked findings: none.

Tool calls:
1. grep for `carriesOver`, `countEqualPositions` and `2*unchanged` across all `.go` files, because my context lacked a repo-wide search.
