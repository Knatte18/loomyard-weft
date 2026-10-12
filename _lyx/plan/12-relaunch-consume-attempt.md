# Card 12 — relaunch-consume-attempt

**Create:**
- `internal/loomcli/arbiterconsume.go`
- `internal/loomcli/arbiterconsume_test.go`
- `plan:internal/loomcli#consumeArbiterAttempt` -> `func consumeArbiterAttempt(attemptDir string, base shedengine.RelaunchRequest) error`
- `plan:internal/loomcli#loomCLI.loomConsumeAttempt` -> `func (c *loomCLI) loomConsumeAttempt(ctx context.Context, attempt string) error`

**Edit:**
- `internal/loomcli#loomCLI.specFor`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedengine#Relaunch`
- `plan:internal/shedverbs#ArbiterAttemptFlag`
- `plan:internal/loomengine#ReadArbiterAttempt`
- `plan:internal/loomengine#MarkArbiterApplied`

**Intent:**
Consume an arbiter's `resume` verdict when the driver re-launches the loop naming its attempt (D1, D6, D7).

`consumeArbiterAttempt` reads the attempt record and verdict in `attemptDir` and refuses, before taking any lock, an attempt whose verdict is missing, unparseable or not `resume`, and one already applied.
It then calls `shedengine.Relaunch` with `base` (status and lock paths, producers, run id), the verdict's goto target, and an `Admit` that, under the status lock, refuses unless the run is in the state the arbiter left: its stop key is the attempt's, `pause_requested` is unset, and `pause_before` and `pause_after` equal the values the attempt recorded at spawn.
`Admit` then writes the applied mark with `loomengine.MarkArbiterApplied`, before `Relaunch` performs the goto, so the mark and the move land under one status-lock hold.
Each refusal names the verdict path, and its way forward is the hand-on path: the driver's stop report quotes the verdict and the run's parent decides.

A `goto` or a `pause` (plain, `--before` or `--after`) set while the arbiter ran fails `Admit`, so neither the goto nor the step happens and no pause is cleared.
A plain `pause` landing after `Relaunch` returns survives, since the step pauses on `pause_requested` before running a producer.
A busy run lock is refused by `Relaunch` before `Admit`, leaving the attempt unapplied.
Bound: the re-launch moves a run only on a `resume` attempt the arbiter verb wrote and validated, at most once per attempt (the applied mark), only backward, and never over a human's goto or pause.

`loomConsumeAttempt` parses the attempt number (a positive integer, refused otherwise), resolves `shedrun.ArbiterAttemptDir(c.location, c.runID, n)`, and calls `consumeArbiterAttempt` with the request built from `c.shedPaths`, `c.routing.Producers` and the resolved run id.
`specFor` sets `spec.Hooks.ConsumeAttempt` to it inside its `c.location != nil` branch.

`arbiterconsume_test.go` drives `consumeArbiterAttempt` over a temp status file and attempt directory with the real `shedengine.Relaunch`, one table-driven test with a row per refusal (missing verdict, `hand-on` verdict, already applied, stop key moved by a goto, plain pause, `--before` pause, `--after` pause, each leaving the status unchanged and no applied mark) and per success (no target: mark written, status unchanged; backward target: mark written, run moved and paused at the target with an existing `pause_before` kept), plus a row where a plain pause written after the call is still set for the next step.
Raise `internal/loomcli`'s row in the test budget by one.

Add loom-section rows to `contracts/specs/refusal-spec.md` for `step --arbiter-attempt: attempt unusable` (missing, not `resume`, bad number), `already applied`, and `run moved or paused since the arbiter spawned`, each a correctness halt whose way forward is the hand-on path.

**ImpactSummary:** `specFor` fills one more hook; every other step path is unchanged.
