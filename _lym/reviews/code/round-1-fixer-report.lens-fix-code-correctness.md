```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I ran the changed branches by hand and checked the added tests against the defect each one guards.

- **`TestRunTierProcessCap`:**
  - Dropping the forced `exit = 1` leaves `ExitCode` at 0, so the test fails.
  - Dropping the `processCapped` flag fails the `wantCapped` check.
  - Dropping the `|| result.processCapped` clause in `failedOnShortage` returns the shortage marker, because the log holds it and the exit is 1. The test fails.
  - A cap started with no run marker fails the `wantStarts` check.
  - The capped row is deterministic. The command waits on the file that the stand-in `list` writes before it returns pids, so `after` is the only ready branch of the select. `line` is set regardless of cancellation, and `watchdogDone.Wait()` orders that write before the read.
- **`tmuxkit.Main`:** `lymtemp.Remove` returns nil when no run directory exists, so a package whose `TestMain` made none is unaffected. Without the new `Remove` call, the child's run directory would stay under `childTemp/lym-&lt;uid&gt;` and the integration test would fail.
- **`reedcli` window seam:** with `lymBinary` nil, `selfexec.LymBinary` is used and the test binary is refused. `window_test.go` is the only test that builds `windowCmd`, and it now injects the stand-in. The tier-1 run for `reedcli` was green.
- **`ideengine.LymBinary`:** the `TestMain` stand-in is an absolute path, which satisfies `filepath.IsAbs`. The tier-1 `ideengine` and `idecli` runs were green. No other test in `internal` calls the `ideengine` entry points or asserts the stamped path. The one other hit, `fabricengine/gitexclude_integration_test.go`, mentions `.vscode/tasks.json` without calling `ideengine`.
- **`spawnDetachedRemove`:** the only test that sets `suppressWatchdogSpawn: false` is in `watchdog_integration_test.go` (line 441), and it concerns the watchdog respawn, not remove.

Tool calls beyond my context:
1. One `grep` for `suppressWatchdogSpawn` in tests: to see whether any test drives the real `spawnDetachedRemove`, which now refuses a test binary.
2. One `grep` for `windowCmd()` and `"window"` in the `reedcli` tests: to find callers of the changed window path.
3. One `grep` for tests that call the `ideengine` entry points or assert the `.vscode/tasks.json` stamp: to find callers of the changed stamp.
