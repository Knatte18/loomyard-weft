# Card 3 — shedverbs-arbiter-attempt-flag

**Create:**
- `plan:internal/shedverbs#ArbiterAttemptFlag` -> `const ArbiterAttemptFlag = "arbiter-attempt"`
- `plan:internal/shedverbs#TestLoop_ArbiterAttemptConsumedByTheWaiter` -> `func TestLoop_ArbiterAttemptConsumedByTheWaiter(t *testing.T)`

**Edit:**
- `internal/shedverbs#Hooks`
- `internal/shedverbs#stepCmd`
- `internal/shedverbs#TestLoop_RefusalsBeforeAnyStep`
- `internal/shedverbs/doc.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Give `step --until-stop` the flag the driver's re-launch names an arbiter attempt with (D6), so the attempt is consumed in Go before the loop starts.

- `ArbiterAttemptFlag` is the flag's name, declared in `loop.go` beside `UntilStopFlag`.
- `Hooks` gains the field `ConsumeAttempt func(ctx context.Context, attempt string) error`: the arming module's consumption of the named attempt, nil for a recipe that consumes none.
- `stepCmd` registers the string flag, visible, with help text saying the arbiter verb's `relaunch` line carries it.

In `stepCmd`'s `RunE`:

- the flag without `--until-stop` is refused before the in-flight record is written, with `kind: bootstrap`, `trace_file`, an `arbiter_attempt` key holding the flag's value, and a way forward naming the `--until-stop` form;
- on the waiter path (`--until-stop` and no `--loop-detached`) with the flag set, a nil `ConsumeAttempt` is refused the same way ("this recipe consumes no arbiter attempt");
- otherwise `ConsumeAttempt` runs before `runUntilStop`; an error wrapping `shedengine.ErrShedBusy` is reported with `spec.StepBusyKind` and every other error with `kind: bootstrap`, its message verbatim, both carrying `arbiter_attempt` and `trace_file`, and no loop is spawned;
- a nil return proceeds to `runUntilStop` as today.

`childArgv` already drops every flag, so neither the detached loop nor its children carry the attempt and none consumes it twice.
The `arbiter_attempt` key is what the driver stencil branches on to tell a refused consumption from any other error envelope.
Bound: the flag can only move a run through the arming module's hook, which card 12 bounds; with no hook it is refused.

`TestLoop_RefusalsBeforeAnyStep` gains four rows, with a column for the step's arguments and the hook since its existing rows run detached: the flag without `--until-stop` (`kind: bootstrap`, the hook never called), and on the waiter path a nil hook, a hook returning a busy error and a hook returning another error, each asserting its kind and error, no child spawned, and in addition `arbiter_attempt` in the envelope and the hook called at most once.
`TestLoop_ArbiterAttemptConsumedByTheWaiter` drives `stepCmd` over the loop fixture of `loop_test.go` for the one path no refusal test covers: a hook returning nil (called once with the flag's value before the first child, loop runs to its stop, the spawned argv carries no `--arbiter-attempt`).
Raise `internal/shedverbs`'s row in the test budget by one.

Add two rows to the shed section of `contracts/specs/refusal-spec.md`: `step: --arbiter-attempt without --until-stop` and `step: recipe consumes no arbiter attempt`, each a correctness halt quoting its way forward.
Describe the flag and the hook's place in the waiter sequence in `doc.go`.

**ImpactSummary:** one new optional flag and hook field on `step`; every existing invocation is unchanged.
