```yaml
lens: fix-affected-set
outcome: reported
```

no findings

The search covers the findings-path literal and the helper's callers. The path is now built once, at `internal/loomcli/prfindings_integration_test.go:101`. The only caller of `blockingFindingsRun` is the subtest at line 63, and it takes both return values. The other `pr-findings.md` hits are a CLI usage string (`prfindings.go:329`) and an unrelated arg-parsing row (`arm_seed_test.go:60`). Neither is a duplicate of the test's path.

Unchecked: none.

Tool calls:
1. grep for `pr-findings.md` and `blockingFindingsRun` under `internal/`, because my context held only the one file and not the other packages.
