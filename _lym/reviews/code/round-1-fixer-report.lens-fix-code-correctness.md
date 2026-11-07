```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes for correctness (F1, F2, F4, F5, F7).

No findings.

Each change, run on one input:

- **F5, `sleep 30 &amp; echo up; wait`:**
  - `sh -c` forks the background `sleep` before `echo up`, so the first write means a child exists.
  - The cancel sends SIGTERM to the whole process group (`internal/verifyrun/run.go:43`), so the backgrounded sleep dies with the shell and the test's timing bound holds.
  - On Windows, `ping -n 30 127.0.0.1` writes its banner only once it runs, so the first write still means a running child.
- **F7, `guardScanRefusal`:**
  - Dropping `%v` and `err` leaves `err` used in the other branches, so the function compiles.
  - The only integration assertion on this refusal is `other/o_test.go:3` plus `way forward`, and both are still produced.
  - No other test pins the old text. The `cmd/testtiming` matches are a different message.
- **F4, `ArmDefaultDeadlineForTest`:**
  - `defaultArmDeadline = armDeadline` is a package-level initialiser, so Go's init ordering resolves it before use.
  - The captured default is unaffected by `SetDeadlineForTest` replacing the variable.
  - The new row fails if the default were a bare `WithCancel`: the context never ends, so the 10 s guard hits `t.Fatal`.
- **F2, `TestScanGuards_DropsWatchedGuardsNoReadChangeConcerns`:**
  - The row `ChangeOutsideTheNestedModuleDropsIt` (`mod`, `p/x_linux.go`) would keep `TestWatchedGuard` without `moduleRelative`'s prefix trim, because `*_linux.go` matches the base name. The test therefore fails with the defect in place.
  - The root-module rows pin the `prefix == "."` path.
  - `UnreadChangesKeepEveryGuard` pins the `changesRead` gate.
  - The gate run passed.
- **F1, the `--card` watch subtest:**
  - Step one dirties the tree, so no green carry applies, and expects `-skip ^(TestWatched)$`.
  - Step two adds `watched/x_linux.go` and expects no skip.
  - The prime step has no parent branch, so `cardChanges` reports false and nothing is skipped. It would fail if `sc.skip` were always filled.
  - The cleanup removes the prime's plan directory and its green file, which fixed the earlier `list -f` interference with "the prime with no parent runs whole". The final gate run had 0 failures.

Calls made:
1. `grep` for "not directly above a top-level" across the repo, to see whether any test pins the old `GuardMisplaced` refusal text, which my context lacked.
2. `grep` in `internal/verifyrun` for the kill path, to confirm the cancel signals the whole process group, which my context lacked.
