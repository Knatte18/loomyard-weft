# Card 8 — verifytree bounds the verify with a timeout

**Edit:**
- `internal/verifytree/verify.go`
- `internal/verifytree/verify_integration_test.go`
- `internal/verifytree/doc.go`
- `internal/loomshed/verifygate.go`
- `internal/websterengine/verifygate.go`
- `internal/landingshed/verifygate.go`
- `internal/webstercli/verify.go`
- `internal/loomshed/verifygate_integration_test.go`

**Uses:**
- `internal/verifyrun/run.go`

**Intent:**
`verifytree.Verify` takes the timeout as a `time.Duration` parameter after the command, and derives its own context from the caller's with it (`context.WithTimeout`), so an expiry stays distinguishable from a caller's cancel (D3).
`verifytree` declares the one production value, `const Timeout = 60 * time.Minute`; a test passes a short value.

When the run fails:

- the caller's own context is done: a returned error and no record, as today;
- otherwise the derived context's deadline passed: a `StatusFailed` result with `ExitCode` -1, the new `Result.TimedOut` set, and `Detail` naming the timeout (for example `the verify command did not finish within 1h0m0s and was killed`); no record is written and the marker is removed by the existing defer.

Bound: the timeout only shortens a verify that would otherwise run unbounded; a timed-out run is a failure, so it lets no failing tree pass.

Every production call site passes `verifytree.Timeout`, mechanically:
`loomshed`'s `NewVerifyGate`, webster's gate seam closure in `NewVerifyGate`, `landingshed`'s `newVerifyGate` (as a closure over the existing seam, so the seam's signature and its test fakes stay as they are) and `webstercli`'s `verify` verb.
How each site words a timed-out result is card 9.
`loomshed`'s integration test calls `verifytree.Verify` directly to seed a record, so it gains the argument too; it and `verify_integration_test.go` are the only test callers.

`verify_integration_test.go` gains the two cases the acceptance criteria name at this layer: a command that outlives a short timeout returns `StatusFailed` with `TimedOut`, writes no record and leaves no marker; a caller-cancelled context still returns an error and writes no record.
`doc.go` states the timeout and the two outcomes.

**ImpactSummary:** One signature change with four production call sites, each passing the new constant; no site's behaviour changes before a verify runs past 60 minutes.
