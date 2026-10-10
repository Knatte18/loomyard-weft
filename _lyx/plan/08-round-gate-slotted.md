# Card 8 — round-gate-slotted

**Edit:**
- `internal/loomshed#NewVerifyGate` -> `func NewVerifyGate(anchorPath, worktreeRoot, verifyDir, siteLabel string, slots *gateslot.Pool) shuttleengine.Gate`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/loomcli#loomCLI.wire`
- `internal/loomshed/verifygate_integration_test.go`
- `internal/webstercli/verify_parity_integration_test.go`
- `internal/loomcli/wiring_test.go`

**Uses:**
- `internal/hubgeom#GateSlots`

**Intent:**
Slot the Webster-Burler round gate (D4).
`loomshed.NewVerifyGate` takes the pool and passes it to `verifytree.Verify`.
`shedrecipe.Env` gains a `GateSlots *gateslot.Pool` field, documented as the hub pool every Go-side verify a row runs acquires from, nil where no hub is wired.
`resolvePlainGateClosure` passes `env.GateSlots` to `NewVerifyGate` for the `verify` gate entry.
`loomCLI.wire` sets `c.env.GateSlots` from `hubgeom.GateSlots(location)`.
`verifygate_integration_test.go` and webstercli's `verify_parity_integration_test.go` pass nil on their existing calls; `TestWire_DefaultConfig` gains a step asserting the env's pool directory is the board dir's slot directory.

**ImpactSummary:** The round gate's one constructor gains a parameter with one production caller and one test file; the round gate now waits for a hub slot.
