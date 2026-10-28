```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1/F2 fix to `gitBranchReadsOnly`'s option tables and the test rows added.

Every walk-through (`git branch -v foo`, `--verbose foo`, `-vv foo` mutating; `-v`, `-v -l feat*`, `--format -x`, `--points-at -x`, `--`, mutating `foo --` as expected) gave the correct result.
The `-v foo`, `--verbose foo` and `-vv foo` rows fail with the old tables; the `--format -x` and `--points-at -x` rows fail without the value skip; the `--`, `-v` and `-v -l` rows are regression guards.
Only `gitInvocationMutates` calls the function and its signature is unchanged.

No findings.

Tool calls: none; my context already held every changed file and test row.
