# Card 5 — Shorten verifyrun's wait delay and verifytree's sleeps in tests

**Edit:**
- `internal/verifyrun#Run`
- `internal/verifyrun/run_integration_test.go`
- `internal/verifytree/verify_integration_test.go`

**Uses:**
- `internal/verifyrun#waitDelay`

**Intent:**
verifyrun's `waitDelay` becomes a per-call seam (D5).
`Run` keeps its signature and delegates to a new unexported `run` that takes the wait delay as a parameter;
`Run` passes `waitDelay`, so both production callers (`internal/verifytree` and webster's card-verify rerun) keep the 10 s bound.
`TestRun_Cancellation` (10.3 s in the measured run) calls `run` with a sub-second delay and tightens its elapsed bound to a small multiple of that delay, so it still proves a cancelled run returns `(-1, context.Canceled)` without warning, now without waiting out the production delay.
The remaining `Run` tests keep calling `Run`, which still pins the public path.

In `verify_integration_test.go`, `TestVerify_CancelledRunLeavesNoRecord` runs `true` instead of `sleep 5` in both its calls (D5): the cancelled call fails before the command starts, and the follow-up call needs only a passing command.

Bound: the seam changes only how long a test waits for a lingering pipe holder; production reads the same `waitDelay`.

**ImpactSummary:** verifyrun's exported `Run` is unchanged in signature and behavior; the edits are internal plus two integration test files.
