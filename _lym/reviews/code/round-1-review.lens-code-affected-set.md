```yaml
lens: code-affected-set
outcome: reported
```

Task restated: for each change, find the set it forces outside the touched lines (lens code-affected-set).

**Per change, the set found:**

- `startCostBound` 100ms to 2s (`lifecycle_integration_test.go:1417`): its only use is the check at lines 1467-1468, which the diff touched, so that is covered. **Not covered:** line 313 is a `//testtiming:keep` marker that still says the step pins "a Command row costs under 100 ms". The bound is now 2 s, so the marker is stale (F1).
- `standInChildSleep` 5s to 5min (`testmain_test.go:44`): used at line 119 for the child's sleep, and in the `runner_test.go` row, which the diff changed. The row now uses `within: time.Minute`. The comment at `testmain_test.go:59` is generic and still correct. Covered.
- `PATTERN-test-speed.md`: new bullet under "Time is injectable". `PATTERN.md:54` is the one-line summary. It is a catch-all and does not list the bullets, so it needs no edit. The commit message has no `Pattern-Departure` trailer, and no declared PATTERN changes were in the directive I saw. Covered.
- Remaining test files: `sleep 300`, `ReexecLockWait`, `helperClaimBoundEnv=10m`, `ping -n 120`/`ping -n 300` and the `elapsed > time.Minute` checks are each local to their own test. The context shows no other reader or fixture. Covered.
- Removed `time` imports (`reedengine`): the compiler is the check, and it is the verify gate's subject.

**Findings:**

- F1 | LOW | consistency | `internal/battencli/lifecycle_integration_test.go:313` | The `//testtiming:keep` justification still cites "under 100 ms", but `startCostBound` is now 2 s. Fix: reword the marker to the new bound, or to the bound's purpose (a Command row that waits seconds on a remote or a fixed delay). Evidence: lines 313 and 1417 of that file.

**Tool calls:**
1. grep of `standInChildSleep`, `startCostBound` and "100ms" in `internal/testcover` and the battencli lifecycle test, plus the `test-speed` line of `PATTERN.md`: the context did not show other users of these symbols or the marker at line 313.
