# Card 13 — gate-entry-expiry-probe

**Create:**
- `plan:internal/shuttleengine#Run.probeExpiry` -> `func (run *Run) probeExpiry() (string, bool)`
- `plan:internal/shuttleengine#TestGatePending_ExpiryProbeEndsMidTurnWait` -> `func TestGatePending_ExpiryProbeEndsMidTurnWait(t *testing.T)`

**Edit:**
- `internal/shuttleengine#GateEntry`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D2, shuttle side: a gate entry's wait expiry is checked off the boundary, so the wait bound ends the wait even when the writer never ends its turn, including after a driver restart.

`GateEntry` gains an optional `Expired func() (bool, error)`, a read-only probe reporting whether the entry's wait has run out; nil means none.
Its doc says the probe sends, counts and writes nothing, that only an entry whose `Attempts` is above zero is probed, and that the loop calls it only while the writer is not at an idle boundary.

`Run.probeExpiry` calls the probe of every entry that has one and is not off, in list order, and returns the first entry whose probe reports expired.
A probe error logs a Warn naming the entry and the error and counts as not expired, so the loop keeps polling under its deadline and liveness checks.

In `Run.Wait`, on a tick whose events read succeeded and which handled no arrival, boundary or restored boundary, while `gateAtBoundary` is false, the loop runs `probeExpiry` once `gateRecheckEvery` has passed since `lastGateEvalAt`, and stamps `lastGateEvalAt`.
It does so whether or not the loop has seen the entry pend, so a loop that attached to a writer already mid-turn after a driver restart, with no boundary seen, still reaches it.
On an expired entry it logs an Info naming the entry, clears the gate memo and returns `finalize(OutcomeDone)`.
Finalize's own evaluation then reads the entries once in list order with no re-prompt, through each entry's `Final` where it has one.
When an earlier must-pass entry fails over the mid-turn tree, finalize stops there and never reaches the expired entry's `Final`, so the outcome is a failing gate and the producer's Stuck.

Bound: the probe ends only a wait whose own entry reports it run out, at most once per `gateRecheckEvery` and never while the writer is at an idle boundary, where the entry's own closure handles its expiry.
The run deadline is never extended.
`finalize` removes the writer's strand like every `OutcomeDone`, so the next start is a fresh spawn.

The finalize call is a positive outcome, so it needs no `allOutputFilesExist` check (`PATTERN-completion-signal`).
`wait.go`'s file header, which counts the finalize call sites, names this one, and `doc.go`'s gate paragraph describes the probe.

Tests:
`TestGatePending_ExpiryProbeEndsMidTurnWait` is a table over `newPendingFixture`-built runs with two entries, a must-pass `discussion` stand-in and a `MayHold` `parent-review` stand-in carrying `Gate`, `Final` and `Expired`:
- after the pending send moved the writer off the boundary, a probe that turns true ends the run `done` within one `gateRecheckEvery` of turning, with `Final` called once and the entry passed when the first entry passes;
- the same with the first entry failing at finalize gives a failing outcome, the second entry `not_reached` and `Final` never called;
- a run attached over an events file holding a turn start and no turn end reaches the probe with no boundary seen;
- a probe that stays false never finalizes before the deadline, and a probe is never called while the writer sits pending at an idle boundary;
- a probe error logs the Warn and keeps polling.
`test-budget.yaml`'s `internal/shuttleengine` row rises by one for the new test.

**ImpactSummary:** one optional GateEntry field and one new finalize call site in the gated wait loop; an entry without a probe, and an ungated run, behave as before.
