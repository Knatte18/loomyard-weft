# Card 12 — relaunch-consume-attempt

**Create:**
- `internal/loomcli/arbiterconsume.go`
- `internal/loomcli/arbiterconsume_test.go`
- `internal/loomcli#consumeArbiterAttempt`
- `internal/loomcli#loomCLI.loomConsumeAttempt`

**Edit:**
- `internal/loomcli#loomCLI.specFor`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedengine#Relaunch`
- `internal/shedverbs#ArbiterAttemptFlag`
- `internal/loomengine#ReadArbiterAttempt`
- `internal/loomengine#ReadArbiterEnvelope`
- `internal/loomengine#MarkArbiterApplied`

**Intent:**
Consume an arbiter's `resume` verdict when the driver re-launches the loop naming its attempt (D1, D6, D7).

`consumeArbiterAttempt` reads the attempt record and `envelope.json`, the verb's validated result (card 10 step 6), in `attemptDir`, never the agent's `verdict.yaml`, and refuses, before taking any lock, an attempt whose envelope is missing or unparseable, whose envelope's `verdict` is not `resume` or whose `cause` is non-empty, and one already applied.
It then calls `shedengine.Relaunch` with `base` (status and lock paths, producers, run id), the envelope's goto target, and an `Admit` that, under the status lock, refuses unless the run is in the state the arbiter left: its stop key is the attempt's, `pause_requested` is unset, and `pause_before` and `pause_after` equal the values the attempt recorded at spawn.
`Admit` then writes the applied mark with `loomengine.MarkArbiterApplied`, before `Relaunch` performs the goto, so the mark and the move land under one status-lock hold.
Each refusal names the verdict path, and its way forward is the hand-on path: the driver's stop report quotes the verdict and the run's parent decides.

A `goto` or a `pause` (plain, `--before` or `--after`) set while the arbiter ran fails `Admit`, so neither the goto nor the step happens and no pause is cleared.
A plain `pause` landing after `Relaunch` returns survives, since the step pauses on `pause_requested` before running a producer.
A busy run lock is refused by `Relaunch` before `Admit`, leaving the attempt unapplied.
Bound: the re-launch moves a run only on a `resume` attempt the arbiter verb wrote and validated, at most once per attempt (the applied mark), only backward, and never over a human's goto or pause.

`loomConsumeAttempt` parses the attempt number (a positive integer, refused otherwise), resolves `shedrun.ArbiterAttemptDir(c.location, c.runID, n)`, and calls `consumeArbiterAttempt` with the request built from `c.shedPaths`, `c.routing.Producers` and the resolved run id.
`specFor` sets `spec.Hooks.ConsumeAttempt` to it inside its `c.location != nil` branch.

`arbiterconsume_test.go` holds one table-driven test over a temp status file and attempt directory with the real `shedengine.Relaunch`.
Most rows drive `consumeArbiterAttempt`: a row per refusal (missing envelope, unparseable envelope, `hand-on` envelope, a `verdict.yaml` saying `resume` under an envelope reading `hand-on` with cause `precondition`, already applied, stop key moved by a goto, plain pause, `--before` pause, `--after` pause), each asserting the refusal's message and the absent applied mark; a row per success (no target, backward target), each asserting the applied mark and the run's current producer; and a row where a plain pause written after the call is still set for the next step.
Rows `0`, `-1` and `x` drive `loomConsumeAttempt` on a `loomCLI` with a temp location, each asserting the bad-number refusal and the absent applied mark.
Pause-condition preservation and the unchanged status file on a refusal are `TestRelaunch`'s (card 2), not asserted here.
Raise `internal/loomcli`'s row in the test budget by one.

Add loom-section rows to `contracts/specs/refusal-spec.md` for `step --arbiter-attempt: attempt unusable` (envelope missing or unparseable, not a validated `resume`, bad number), `already applied`, and `run moved or paused since the arbiter spawned`, each a correctness halt whose way forward is the hand-on path.

**ImpactSummary:** `specFor` fills one more hook; every other step path is unchanged.
