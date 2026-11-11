```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness. I traced each new and changed row through `check`, `rerunTestFailure` and `verdict`. I found no defects.

**Per branch, case run, verdict**
- **"a passing verify runs no rerun"**: `results=[passed]`, no sink-independent log. `rerunTestFailure` returns `first` because the status isn't failed. `verdict` passes and the reason is empty. One command, `crashed=false`. Correct.
- **"a failure whose log cannot be read"**: `writeLog` runs `MkdirAll(path)`, so `verify-1.log` is a directory. `ReadFile` errors and `rerunTestFailure` warns and returns `first`. `verdict` gives the "failed after merging" reason naming `result.Log`, which is call 1's log. Correct.
- **"a log with no failure report"**: log is `"FAIL\n"`. `TierRunFailures` finds no header and returns `ok=false`, so there is no rerun. The reason names log 1. Correct.
- **"a package that failed outside any test"**: `reportFailureLine` returns the failure with the `, failed outside any test` suffix, and `TierRunFailures` returns `false` (`runlog.go:390`). No rerun. Correct.
- **"a rerun whose runner crashes"**: `rerunTestFailure` returns the crashed second result. `recordFailure` returns early because `recorder` is nil. `verdict` takes the `RunnerCrash` branch, and the `LandingStopPrefix` reason contains `verify-2.log`. `check` returns `crashed=true`. Correct.
- **"a rerun whose shell could not start"**: `ExitCode=-1`, status failed, and the fake still writes log 2. `verdict` gives the "could not start" reason naming log 2, and `crashed=false`. Correct.
- **"a rerun the verify cannot run"**: `onVerify` sets `verifyErr` on call 2, and `results[min(1,0)]` is `failed`. `rerunTestFailure` returns `Result{}` and the error. `check` wraps it with `%w`, so `errors.Is(err, rerunErr)` holds. The reason is empty, which matches `wantReasonLog=0`. `crashed=false`. The `verifyErr` assignment happens only at call 2, so earlier calls are unaffected. Correct.
- **Rows with a nil `rerunErr`**: `errors.Is(err, nil)` is true only when `err==nil`, so the assertion stays strict. Correct.
- **"a runner crash is not rerun"**, with the new `wantCrashed` expectation: `first.RunnerCrash` returns early. `check` returns `Status==Failed && RunnerCrash`, which is true. This matches `rerunTestFailure` and `verifygate.go:285`. Correct.
- **`TestNewFinalize_WiresTheGateFlakyPassesIntoTheLandingTexts`**:
  - **Result indexing:** `min(verifyCalls, len)-1` is valid, since `verifyCalls` is at least 1 when `onVerify` runs.
  - **Wiring:** `fz.gate` is a value field mutated in place, so `onFlakyPass`, bound to `fz.noteFlakyPass`, survives the seam replacement.
  - **Pass path:** the rerun passes and `onFlakyPass` appends to `fz.flaky`. `recordFailure` returns early on a pass. `clearFailure` runs `RemoveLandingFailure` on the fixture paths and is harmless.
  - **Failure mode:** without the `assembleFinalize` wiring, `onFlakyPass` would be nil, there would be no rerun, and `check` would return a failure reason. The test fails, so it does pin the wiring.
  - **Unverified:** whether `NewFinalize` succeeds with a zero `&fabricengine.Fabric{}` depends on `newTestDeps` and `mergeresolve.New`. I did not check this, but the test fails loudly via `t.Fatalf` if it doesn't.

**Tool calls**
1. Read `gateslot/runlog.go` lines 360-420 and `publish_verify_test.go` lines 8-30. My context lacked `TierRunFailures`' guard semantics and the `writeVerifyLog` body.
