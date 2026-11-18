```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- Go 1.26 gives each loop iteration its own `name`, so the parallel subtests do not share it.
- Each subtest builds its own fixture with the subtest's `t`, so nothing is shared between them.
- With the defect in place, a name dropped from `optionalPullRequestSeams` makes `checkWiring` refuse the unset seam. `Call` then returns an error and that row fails, because the names are now literal in the test.
- A renamed field would panic in `FieldByName(...).SetZero()`, which also fails the row.
- The production code is unchanged, so no caller is affected.

Tool calls:
1. Read `go.mod`'s `go` line, because my context lacked the Go version needed to judge loop-variable capture.
