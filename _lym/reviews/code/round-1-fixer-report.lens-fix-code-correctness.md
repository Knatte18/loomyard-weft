```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes for correctness (`installPauseGate`, the new `TestInstallPauseGate_HoldsOnlyARunsOwnVerbs`, the `shuttleengines_test.go` constant swap).

No findings.

- **`installPauseGate` extraction** (`cmd/lym/main.go`): It passes the same four arguments `InstallNetworkGate` took before. Its only call site is the root pre-run, and `tendercli` stays imported through `InstallOwnerPause`, so no caller changes. A concrete run of `board get` goes `runCommand(cmd)` false, then `heldByPause` false, then `ExemptAll`, so the hub entry passes. For `shed run`, the mark is set, `heldByPause` is true, and the hub entry covers the gate.
- **New test, mutation check:** If the root dropped `runCommand(cmd)` or passed `false`, the `shed run` and `beam dobby` rows read `GateCovered() == false` and fail. If it passed `true` always, the `board get`, `machine name` and `beam start` rows fail. The new test therefore fails with the defect in place, which the review's F2 asked for.
- **New test, isolation:** The parent test `t.Setenv`s `LYM_RUN_PROCESS` and every row does too, so the `os.Setenv` inside `InstallNetworkGate` is undone and a mark can't leak into the next row. The sentinel gate's `t.Cleanup` restores the gate state after the per-row installs. The rows are not parallel, and the test says why.
- **F3 test:** The environ entry and the scrub case now use `pauserecord.RunProcessEnv`. If the constant is renamed, `EnvScrub` drops the new name and the kit's literal keeps it, so `ServerEnviron` and `want` differ and the test fails. A cycle through the new `pauserecord` import is ruled out: the `go vet` of `./cmd/lym` and `./internal/shuttleengines` was clean, and the tier 0 and tier 1 gate passed with `TestInstallPauseGate_HoldsOnlyARunsOwnVerbs` running.
- **F1 doc:** The text matches `heldByPause` in `internal/tendercli/gate.go` and the `beam dobby` row of `TestRunCommand_MarksOnlyARunsOwnVerbs`.

Further tool calls: none, because the context already held every changed file and the gate output.
