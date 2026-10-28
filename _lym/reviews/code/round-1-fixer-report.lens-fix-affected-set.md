```yaml
lens: fix-affected-set
outcome: reported
```

Fix-affected-set: no findings. The fix covers every member of both sets.

Unchecked findings: none.

- **F1:** the only `t.Run` calls indented two or more tabs in `internal/lenses/*_test.go` are loop-driven row runners (lines 94, 119, 474, 626); none wraps a named sibling subtest.
- **F2:** the only uses of `procedureLenses[` are at lines 391 (population), 401 (the set) and 449 (the remaining `leanness` check). No other assertion reads a value the test just set.

Tool calls: 1 `grep` over `internal/lenses/*_test.go`.
