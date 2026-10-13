# Card 8 — tackler-consume

**Create:**
- `internal/loomcli#consumeTacklerAttempt`
- `internal/loomcli/tacklerconsume_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomcli#consumeArbiterAttempt`
- `internal/loomengine#ReadTacklerAttempt`
- `internal/loomengine#ReadTacklerEnvelope`
- `internal/loomengine#TacklerApplied`
- `internal/loomengine#MarkTacklerApplied`

**Intent:**
D7: a tackler `resume` is performed through the arbiter's consume path, so the mark and the move land under one hold of the status lock and a restarted driver can tell a `resume` never performed from one whose stop key returned.

A new file `internal/loomcli/tacklerconsume.go` holds `consumeTacklerAttempt`, shaped like `consumeArbiterAttempt`.
It reads the attempt record and the validated envelope, never `verdict.yaml`.
Before taking any lock, it refuses an envelope that is missing or unparseable, one that is not a `resume` or carries a cause, and an attempt already applied.
It then calls `shedengine.Relaunch` with the envelope's `goto` as the target and an `Admit` that, under the status lock, refuses unless the stop key equals the attempt's and the pause conditions equal those the attempt recorded at spawn, and writes `loomengine.MarkTacklerApplied` in the same hold.
Every refusal names the attempt directory and the way forward: the driver acts on the refused stop as a new stop (D12), or reports it in the stop report and the run's parent decides.
A `resume` with no `goto` relaunches with an empty target, which leaves the status unchanged, as the arbiter's does, and the driver's next `RunUntilStop` re-steps.

Test `TestConsumeTacklerAttempt` in `tacklerconsume_test.go`, a table mirroring `TestConsumeArbiterAttempt` over a real status file in a temp dir:
- an admitted `resume` with a goto moves the run and leaves the applied mark;
- one without a goto leaves the status as it was and leaves the mark;
- a stop key that moved since the spawn, and a pause condition added since, each refuse with no mark and no move;
- an already applied attempt, a `hand-back` envelope and a missing envelope each refuse before any lock.
`test-budget.yaml`'s `internal/loomcli` row rises by one.

**ImpactSummary:** one new unexported function in `internal/loomcli`, called by the Go driver from card 13; the budget row is the only existing file touched.
