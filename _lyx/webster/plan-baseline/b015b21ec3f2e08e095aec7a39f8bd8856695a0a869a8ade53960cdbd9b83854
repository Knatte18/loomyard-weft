# Card 7 — landing-slotted

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#newVerifyGate`
- `internal/loomcli#landingDeps`
- `internal/loomcli/landingdeps_verify_test.go`

**Uses:**
- `internal/hubgeom#GateSlots`

**Intent:**
Slot Publish's plan verify and `publish_verify` and Finalize's plan verify (D4).
`landingshed.Deps` gains a `GateSlots *gateslot.Pool` field, documented as the told hub pool both producers' verifies acquire from.
`newVerifyGate` passes it to `verifytree.Verify` for every verify the gate runs.
`landingDeps` fills it with `hubgeom.GateSlots(l)`.
`landingdeps_verify_test.go` asserts the wired pool's directory is the hub board dir's slot directory, beside its existing verify-dir assertion.

**ImpactSummary:** Publish and Finalize now wait for a hub slot before each verify; their verdicts and Stuck reasons are unchanged.
