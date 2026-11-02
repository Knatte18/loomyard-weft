```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each as-proposed fix (F1, F3, F4, F5, and the proposal behind F2) for a case where it is wrong or breaks something.

No findings.

- **F1:** `var defaultStallTimer = startStallTimer` is safe under Go's package-level init-dependency ordering. It captures the production closure before any test installs the seam. The 1 ms row gives `stopFired()` false, because `Timer.Stop` reports false once the timer has expired. The one-hour row gives `stopPending()` true. The pending check can never fire, so its `t.Error` cannot run after the test ends.
- **F2:** The fixer used `firedAt` instead of the proposed per-attempt baseline, which would have been wrong in one case. A baseline taken at the first `ready` call misses a script run that lands before the first 5 ms tick. `firedAt` has no such case. An EAGAIN-retried arm leaves `firedAt` unchanged, so the readiness test is independent of the attempt number. Each attempt's `ready` goroutine stops before the next one arms, so the `atomic.Int64` is enough.
- **F3:** `checkRan` is set in the same goroutine and step that returns true to the seam, immediately before `check()` runs. If the trace stays empty, the flag stays false and the new assertion fails.
- **F4:** Dropping `armed()` loses nothing. If the check were wrongly armed, the always-true `ready` would fire at the first tick, find an empty trace and cancel the attempt. `Stalled` would then be true and `runs` would be 2, so both existing assertions fail.
- **F5:** The comment edit is accurate and breaks nothing.

Tool calls: none; the context already held every changed file.
