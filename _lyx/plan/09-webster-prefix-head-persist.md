# Card 9 — webster-prefix-head-persist

**Edit:**
- `internal/websterengine#State`
- `internal/websterengine#verifyGateSeams`
- `internal/websterengine#NewVerifyGate`
- `internal/websterengine#newVerifyGate`
- `internal/websterengine/doc.go`
- `internal/websterengine/verifygate_test.go`
- `internal/websterengine/state_test.go`

**Intent:**
D6's `pre-fix` target: the verify gate's pre-fix head lives only in the gate closure's memory today, so no later verb can reset to it.

`State` gains `PreFixHead string` (`json:"preFixHead,omitempty"`), documented as the HEAD the latest gate run started its fixes from.

`verifyGateSeams` gains `recordPreFix func(head string) error` and `clearPreFix func() error`.
`NewVerifyGate` wires both to load state.json under `AcquireStateMutation`, set or clear `PreFixHead`, and save; Master is idle at a gate evaluation, so no bracket verb holds the lease then.
`newVerifyGate` keeps its per-shuttle-run semantics (attempt count and the closure's own `preFix`) and adds:
at the closure's first failed evaluation in this shuttle run, the moment it sets `preFix`, it calls `recordPreFix`, overwriting any value an earlier run left;
on a passing evaluation it calls `clearPreFix`.
A persist failure is returned as the gate's error, like the closure's other seam errors.
`run --fresh` archives state.json, which clears the value with it; the `reset` verb clears it in a later card.

Tests (fake seams): the first failure records HEAD once and a second failure in the same closure does not re-record;
a new closure's first failure overwrites with its own HEAD;
a pass clears;
state.json round-trips `preFixHead`.
Document the persisted pre-fix head in `internal/websterengine/doc.go`'s verify-gate section.

**ImpactSummary:** the verify gate gains two state writes at points it already reaches; its pass/fail verdicts are unchanged.
