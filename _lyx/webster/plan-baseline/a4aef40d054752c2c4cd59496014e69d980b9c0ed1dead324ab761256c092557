# Card 14 — Integration and live smoke for the parent review

**Create:**
- `internal/loomcli/parentreview_integration_test.go`
- `internal/loomcli/smoke_parentreview_test.go`

**Uses:**
- `internal/parentreview#NewGate`
- `internal/parentreview#Store`
- `internal/parentreview#Store.Latest`
- `internal/loomcli#RunCLIIn`
- `internal/loomcli/smoke_gate_test.go`

**Intent:**
Cover the end-to-end acceptance criteria the unit cards cannot reach.

`parentreview_integration_test.go` (`integration` tag) builds a hub through `hubforge`, seeds a task worktree with a parent name, and builds the Discussion-Write row's real gate list from `wire`.
A scripted shuttle fake replays arrivals against that gate list the way the wait loop does: the first arrival over a valid discussion opens round 1 and carries the delivery prompt naming the parent; the test runs `lyx loom review delivered` in the task worktree and `lyx loom review reject <slug> <file>` from the prime through `RunCLIIn`; the next arrival fails with findings naming the copied `review.md`; after the fake rewrites the discussion, the next arrival passes and is let through without a second review.
It then asserts the request directory is in the discussion commit, `lyx loom status` showed the wait while the request was open, a fresh start after the completed round opens round 2, and an `approve` path lets the run through with no re-prompt.

`smoke_parentreview_test.go` (`smoke` tag, compiled by the plan's verify and run by hand) follows `smoke_gate_test.go`: a hand-started parent session is named as the seed's parent, a live Discussion-Write run reaches the gate, and the test waits for the parent's session to receive the SendMessage notice naming the request path.
