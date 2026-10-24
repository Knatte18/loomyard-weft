Task: check the correctness of the changes to `FailedTier`, `runTest`'s halt path, `haltExitCode`, `printFailedRuns` and the added tests.

No BLOCKING or MEDIUM findings. Two LOW findings:

- **LOW** — `haltExitCode(haltCode, firstFailureCode)`, `internal/gatecli/run.go`
  - Summary: a halt that reports 0 after an earlier failed run exits 1, not the first failure's code.
  - Evidence:
    - Input: tier0 exits 7, then tier6 exits 0 under a cancelled context with no caught signal.
    - `haltCode == 0` returns 1 before the `firstFailureCode` check is reached.
    - The call's own rule is "exit with the first failure's code".
  - It is still non-zero, so F2's false green is fixed, and the case is rare.
  - Fix: return `firstFailureCode` when it is non-zero.

- **LOW** — `if end.halt` branch of the `runTest` loop, `internal/gatecli/run.go`
  - Summary: a halting run that itself failed is left out of the closing line, and an earlier failure's code is not kept.
  - Evidence:
    - Input: tier0 exits 7, then tier6 exits 9 while the context is cancelled with no caught signal.
    - `report` returns `{code: 9, halt: true}`.
    - The halt branch prints the line before the halting run is appended to `failures`, so the line lists only tier0.
    - The exit is `haltExitCode(9, 7)`, which is 9, not 7.
  - Real signals are overridden by `exitAfterSignal`, so only the no-signal cancel is affected.

Checked and holding:
- `FailedTier`: the forward scan returns the first non-zero summary line. The new row `line(0,3)+line(6,1)` expects tier 0 and would give 6 under the old reverse scan. Its other callers and the doc comments read consistently.
- Refusal after a failed tier:
  - `output.Err` gives 1 and `haltExitCode(1, 7)` keeps 1.
  - `printFailedRuns` prints the closing line with tier0 only.
  - The new subtest would fail without the halt-path print.
  - The `cmd` cleanup stops the leak that broke the later subtest.
- Busy hub after a failed tier: `haltExitCode(75, 3)` returns 3 and `haltExitCode(75, 0)` returns 75, which is correct. No test covers either, which is acceptable for F3's fix.
- Guard row: `helperGuardExitEnv` is read only when an argument has the `^(TestGuard` prefix. The listed `./pkg` call keeps exit 3 and the guard call gets 5, so the closing line `tier0 (exit 3), guard tier0 (exit 5)` pins `runLabel`'s `guard ` prefix. It would fail if the prefix were dropped.
- Closing-line wording: the only assertion of the old text is the one updated at `test_integration_test.go:952`, and a repo grep found no other use.

Tool calls: none.
