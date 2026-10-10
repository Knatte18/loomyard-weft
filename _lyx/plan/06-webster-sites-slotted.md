# Card 6 — webster-sites-slotted

**Edit:**
- `internal/websterengine#NewVerifyGate`
- `internal/webstercli#websterCLI.runVerify`
- `internal/websterengine#rerunCardVerifies` -> `func rerunCardVerifies(cards []planparser.Card, worktree string, timeout time.Duration, slots *gateslot.Pool) []string`
- `internal/websterengine#RecordBatch`
- `internal/websterengine/cardverify_test.go`

**Uses:**
- `internal/websterengine#Geometry`
- `plan:internal/gateslot#Pool.Acquire`
- `plan:internal/gateslot#Lease.Env`

**Intent:**
Slot webster's own gate runs (D4).
Webster's verify gate passes `geom.GateSlots` to `verifytree.Verify`, and `lyx webster verify` passes its geometry's pool the same way.
The in-process card `**Verify:**` rerun takes the pool too: for each card with a Verify it acquires a slot with a `Holder` naming the worktree and the site `card verify NN-<slug>`, starts the card's timeout once the slot is held, runs the command through `verifyrun.Run` with `lease.Env(os.Environ())`, and releases the slot; a nil pool runs unslotted with the parent environment, as today.
A failed acquire is a failing card line naming the acquire error, consistent with a verify that cannot start.
`RecordBatch` passes `deps.Geom.GateSlots`.
`cardverify_test.go` (integration tier) passes nil on its existing rows and gains one row with a temp-dir pool whose command fails unless `GOFLAGS` carries the pool's `-p` cap.

**ImpactSummary:** Webster's gate, `lyx webster verify` and record-batch's card-verify rerun now wait for a hub slot; standalone runs stay unslotted.
