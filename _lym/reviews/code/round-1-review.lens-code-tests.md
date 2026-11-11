```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests. Each behaviour the diff adds or changes, and the test that pins it:**

- **`NewFinalize` wires the gate's flaky sink to the Finalize it returns.**
  - Pinned by `TestNewFinalize_WiresTheGateFlakyPassesIntoTheLandingTexts` (`finalize_gate_test.go:249-279`).
  - The test builds a real `NewFinalize` result and swaps in only the gate's command, paths, dirty and verify seams. The sink `assembleFinalize` set stays in place.
  - With the wiring removed, no rerun runs and the second verify never happens. The gate then fails, so `check()` returns a reason and the test fails.
  - With the sink pointing elsewhere, the `landingTexts` assertions fail.
  - It is distinct from `TestFinalize_FlakyGateFailureLands`, which goes through `assembleFinalize`, not `NewFinalize`.
- **A passing first verify runs no rerun.** This is pinned by the new row "a passing verify runs no rerun" (`verifygate_test.go:411`).
- **The first-verify `Status != Failed` guard is not pinned by any row.** This is the finding below.
- **`TimedOut`, `RunnerCrash`, `ExitCode<0` and `Log==""` each skip the rerun.** They are pinned by the existing rows (`:425,427,428,429`). The `ExitCode<0` row works because the fake writes a log for a Failed status.
- **An unreadable log skips the rerun.** The new row at `:430` puts a directory where the log belongs, so `ReadFile` errors. Dropping the `err != nil` return would make the rerun run, and the row would fail on `wantCommands`.
- **A log with no failure report, a build failure and a package that failed outside any test skip the rerun.** The new rows at `:431` and `:432-435` and the existing row at `:424` pin these. Each makes `TierRunFailures` report no test runs.
- **Rerun outcomes:**
  - A pass is flaky: existing row `:413`.
  - A skip over a recorded pass is flaky: existing row at `:452`.
  - A repeated failure is judged on the rerun's log: existing row at `:420`.
  - A rerun timeout is judged on the rerun's log: existing row at `:436`.
  - A rerun runner crash halts on the rerun's log with `crashed` set: new row at `:440`.
  - A rerun whose shell could not start is judged on the rerun's log: new row at `:444`.
  - A rerun verify error becomes the gate's error, with no flaky report: new row at `:448`. The `errors.Is` assertion, the `commands` check and `wantFlaky` nil cover the error wrapping and the no-sink-call case.
- **Changed expectation on a first-run runner crash (`crashed` flag, no rerun).**
  - Pinned by row `:427` with `wantCrashed: true`.
  - It matches the code: `check` returns `Status==Failed && RunnerCrash`, and the first-run crash is not rerun.
  - The new `crashed != tt.wantCrashed` assertion applies to every row, so the other rows pin `crashed == false`.
- **Test economy.** I found no duplicates. The two new tests are marked "not judged" in the redundancy report, and I found no existing test covering what they pin.

**Findings:**

1. **LOW, class design, `internal/landingshed/verifygate_test.go:411`.**
   - The "a passing verify runs no rerun" row does not pin the `first.Status != verifytree.StatusFailed` guard in `rerunTestFailure` (`verifygate.go:296`).
   - The row sets no `log`, so the fake gives the passed result an empty `Log`. The `first.Log == ""` guard in the same condition then returns early anyway.
   - Deleting the status guard would leave every row green.
   - No row has a non-Failed first result that carries a readable test-failure log.
   - Evidence: `verifygate_test.go:411` has no `log:` field. In the fake at `verifygate_test.go:83`, `result.Log` is set only when `f.writeLog != nil`, and `writeVerifyLog` runs only when `tt.log != ""` (`:462`).
   - Fix: give the row `log: goFailure`, so the passed result carries a failure log, and keep `wantCommands: []string{gateLandingCall}`.

**Tool calls:** none made. The manifest already held both test files, `verifygate.go` and the fake.
