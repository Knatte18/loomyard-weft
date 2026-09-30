# Card 13 — loomcli-approve-at-gate

**Edit:**
- `internal/loomcli/approve.go`
- `internal/loomcli/approve_test.go`
- `internal/loomcli/approve_awaiting_test.go`
- `internal/loomcli/cli.go`

**Uses:**
- `internal/landingshed/rejection.go`
- `internal/loomshed/loomshed.go`

**Intent:**
Move `approve` onto the gate row and make the latest operator decision win (D4).
`approveVerb` accepts a run awaiting or blocked at `loomshed.NamePRGate` and refuses every other producer, `Publish` included, with the existing message shape naming the gate.
`approveDeps` gains `removeRejection func() error`;
after every check passes, the verb calls it before `writeApproval`, so a crash between the two leaves no record rather than both.
The cobra wiring fills it with `landingshed.RemoveRecord(loomengine.LoomRejectionPath(location))`.
Update the file comment, `Short`, and `Long` to name the gate row instead of Publish, and `cli.go`'s parent `Long` sentence about `approve` the same way.
Tests: `approve_awaiting_test.go` asserts a run awaiting and one blocked at the gate are accepted, and one awaiting at `Publish` is refused;
`approve_test.go`'s success case asserts the rejection record is removed before the approval is written (call order recorded by the fakes), and a `removeRejection` failure refuses with one error envelope and writes no approval;
the help test names the gate.

**ImpactSummary:** Changes approve's accepted row from Publish to the gate; no other verb reads that check.
