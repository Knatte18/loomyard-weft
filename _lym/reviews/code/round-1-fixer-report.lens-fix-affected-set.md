```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check, rerunning each set-premise finding's search.

**Unchecked findings:** none. I reran F1, F2, F4 and F5. F3 is the parent's to rule on and the fix did not touch it.

**Set coverage**
- **F1:** `SetSpawnBoardSyncForTest` has one caller in the package (`testmain_integration_test.go:35`), and it now passes through to `boardengine.SpawnSync`. No other board-sync stand-in exists there.
- **F2:** both `EnsureOrWarn` calls in tests are in the restored test (`ensure_integration_test.go:667`, `:683`). The summary's "Retry walks removed" bullet no longer lists it.
- **F4:** the summary states the line count once (`summary.md:20`), and 390/428 matches `git diff --stat 2326ef4da..HEAD`.
- **F5:** besides the logger-capturing tests, the rows that run alone are the no-notifier row (`reinterpret_integration_test.go:69`) and `FailingSeam`. The correction in the fix commit names both.

**Findings**
- **LOW. The restored second walk, against the "Timings" section of `_lym/landing/summary.md`.** The after figures (3.65 s test binary time at line 15, 9.6 s wall at line 19) were measured before the fix added a second `EnsureOrWarn` walk to a serially run row. The section now reports a run of a test set that no longer matches the change. I did not measure how large the difference is.
- **LOW. The F2/F4/F5 commit message, against the body of `f2ebdb2c1`.** That body says "two reinterpretation rows and three retry walks repeat another test's path". With the `EnsureOrWarn` retry restored, only two of those retry walks stay removed. The fix commit's correction covers the rows that run alone but not this count.

Outside my lens: `docs/research/no-fake-other-modules/fabric-batten.md:82-83` still names `recordNotices` and `recordBoardSyncs`. The earlier diff made those names stale, not this fix.

Tool calls (4 of 6):
1. Grep for board-sync seams in `internal/hubreconcile`: my context had no package-wide search for seam uses outside the files read.
2. Grep for `EnsureOrWarn(` in tests under `internal` and `cmd`: the earlier search covered only `internal/hubreconcile`.
3. Grep for the line count under `_lym/landing` and `_lym/loom`, plus `git diff --stat`: my context had not searched for other statements of the figure.
4. Grep for serial and parallel markers in `reinterpret_integration_test.go`: that file was not in my context.
