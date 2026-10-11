# Card 12 — gate-recheck-throttle

**Create:**
- `plan:internal/shuttleengine#gateRecheckEvery` -> `const gateRecheckEvery = 5 * time.Second`
- `plan:internal/shuttleengine#TestGatePending_IdleRecheckThrottled` -> `func TestGatePending_IdleRecheckThrottled(t *testing.T)`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.handleGatedBoundary`
- `internal/shuttleengine#TestGatePending_VerdictBetweenTicksIsNotMasked`
- `internal/shuttleengine#TestAttachGated_Pending`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D3: the wait loop re-runs a pending gate at an idle boundary with no new arrival at most once per `gateRecheckEvery`.
At the 500 ms poll every pending tick re-ran `discussionparser.Validate`, three locked JSON reads and a `MkdirAll`, and a verdict read 5 seconds late costs nothing.

`Run` gains `lastGateEvalAt`, the run clock's time of the last gate evaluation, zero before the first; `handleGatedBoundary` stamps it on entry, before `evaluateGate`.
In `Run.Wait`, the branch for a tick with no new arrival while the writer sits at the boundary runs `handleGatedBoundary` for a pending entry only once `gateRecheckEvery` has passed since `lastGateEvalAt`.
An unsent re-prompt's retry on that branch stays per tick, since it re-sends a re-prompt rather than re-reading a verdict.
A new turn-end arrival and a restored cleared boundary still evaluate at once.
Card 13's off-boundary probes share the cadence and the stamp.

Bound: the throttle only delays a re-read by up to `gateRecheckEvery`; it never skips an arrival, never extends the run deadline and leaves the poll interval unchanged (`PATTERN-daemon-wakeups`).

`wait.go`'s file header and `doc.go`'s gate paragraph say a pending entry is re-evaluated with no new arrival at most once per `gateRecheckEvery` rather than on every poll tick.

Tests:
`TestGatePending_IdleRecheckThrottled` drives `newPendingFixture` with a bare pending gate over several one-second ticks and asserts the closure runs at the arrival and then once per `gateRecheckEvery` of fake-clock time, and that a turn-end arrival appended between rechecks is evaluated on its own tick.
`TestGatePending_VerdictBetweenTicksIsNotMasked` asserts the verdict recorded between ticks is read at the first throttled recheck, with the call count that cadence gives.
`TestAttachGated_Pending`'s bare-pending case asserts re-evaluation on the throttled cadence rather than every poll tick, in its assertions, doc comment and `testtiming:keep` reason.
`test-budget.yaml`'s `internal/shuttleengine` row rises by one for the new test.

**ImpactSummary:** one in-memory throttle inside the gated wait loop's idle re-evaluation; arrivals, re-prompt retries, deadlines and the poll interval are unchanged.
