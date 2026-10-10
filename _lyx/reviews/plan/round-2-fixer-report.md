# Plan-Review round 2 — fixer report

Every finding was validated against the code and the decision record and fixed.
`lyx loom validate-plan` passes the edited plan (`ok: true`, informational `create-new-unit` findings only).

## Fixed

- **F1** (card 6 derives the wait directory inside `websterengine`).
  Confirmed: `websterengine` is a `PATTERN-told-geometry` bound package, and `Geometry`'s doc says webster never derives its paths itself; `VerifyDir` is the established told-field form.
  Card 5 adds `Geometry.GateWaitDir`, filled by `hubgeom.WebsterGeometry` with `gateslot.WaitDir(anchorPath)` and left empty by `standalonegeom`; `plan:internal/gateslot#WaitDir` joins card 5's `Uses:`, and `webstergeom_test.go` asserts the field.
  Card 6's `RecordBatch` passes `deps.Geom.GateWaitDir`, and `gateslot#WaitDir` leaves card 6's `Uses:`.
  Card 9's `loomCLI.wire`, `wireHub` and `wireStandalone` read `GateWaitDir` from the webster geometry in place of joining again; `wireLightweight`, which has no webster geometry, keeps its CLI-layer `gateslot.WaitDir` join.
- **F2** (card 22's Bound omits a tampered record's effect on the review).
  Card 22's Bound now states D11's Bound for the review: deleting the record skips the widening; after `checkedPublishFailure`'s shape checks, editing it can only add a well-formed test name that never failed, forcing a BLOCKING finding, or point the reviewer at another real merge commit's diff; no shell text or free prose reaches the prompt, and Publish reruns both verifies regardless.
- **F3** (stale FAILED condition in `internal/websterengine/doc.go`).
  Confirmed at `internal/websterengine/doc.go:70`.
  Card 15, which already edits that doc, now also restates the fork-return sentence: FAILED on a failed card gate, card `**Verify:**` or batch gate, never on deviation alone and never on a slot-busy exit.
- **F4** (status names only one waiting gate).
  Card 11's `gateWaiting` now renders every live wait record, oldest first, with the holders once, and appends the rest of the chain's note after its own, so a live verify marker never hides a gate wait.
  The chain order becomes gate wait, verify, shuttle, then parent review.
  The chain test rows replace the single gate-wait row with two live records rendering both sites and a wait record beside a live verify marker.

## Disputed

None.

## Deferred

None.
