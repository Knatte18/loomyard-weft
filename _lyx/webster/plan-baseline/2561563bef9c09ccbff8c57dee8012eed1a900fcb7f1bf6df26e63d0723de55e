# Card 16 — The loomcli smoke tier passes

**Edit:**
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/smoke_driverstrand_test.go`
- `internal/loomcli/smoke_drivingsurface_test.go`

**Uses:**
- `internal/loomcli/bootstrap.go`

**Intent:**
D12: fix the failing smoke tests at their cause, confirmed by running the tier, not assumed.

First run `go test -tags smoke ./internal/loomcli/` and read each failure.
The working hypothesis: the tests are stale.
`statusStrandCount` counts strands by exact name equality with `statusStrandDisplayName` and `driverStrandDisplayName`, while production names strands through `internal/agentname` and finds them with `agentname.Matches` (`findStatusStrand`) and `loomengine.IsDriverStrand` (`findDriverStrand`).
If the run confirms it, `statusStrandCount` matches a status strand through `agentname.Matches`, a driver-strand count goes through `loomengine.IsDriverStrand`, and the call sites in `smoke_driverstrand_test.go` and `smoke_drivingsurface_test.go` move to whichever matcher fits their name.
Diagnose `TestSmokeStart_LLMRunRemovesEveryStatusStrand` the same way.
If a failure shows a production regression instead, fix production in this card, adding that file to the card's targets, and keep the test asserting the intended behaviour.
The tests stay in the `smoke` tier (`PATTERN-test-tier-purity`) and keep `tmuxkit.Main` isolation.

**ImpactSummary:** Smoke-tier test helpers only, unless the run shows a production cause.

**Commit:** 16: The loomcli smoke tier matches strands the way production names them

**Verify:** go test -tags smoke ./internal/loomcli/
