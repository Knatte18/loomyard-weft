# Card 2 — step-after-hook

**Edit:**
- `internal/shedverbs/spec.go`
- `internal/shedverbs/step.go`
- `internal/shedverbs/step_test.go`
- `internal/loomcli/step_test.go`
- `CONSTRAINTS.md`

**Uses:**
- `internal/shedengine#StepResult`
- `internal/shedengine#ErrShedBusy`

**Intent:**
Give the generic `step` body the after-step extension point `run` already has in `PostRun`, and a `friction` key on the step success envelope, so loom can file Tier 1 anomalies, write halt notes and reflect inside the step process before the envelope is written (D2 and the hub-relayed Tier 1 addition).

`Hooks` gains `AfterStep func(ctx context.Context, res shedengine.StepResult, stepErr error) string`.
`stepCmd` calls it once `shed.Step` has returned, on every path: on success after `PostStep`, and on every `shed.Step` error, busy included, before the refusal envelope.
It is not called when `PreStep` or `BuildShed` refuses, since no step ran.
Its doc mirrors `PostRun`'s: the busy error is passed through so the hook applies its own busy skip, and on an error path the hook runs for its side effects only, because a step error envelope carries no extras, exactly as `run`'s does.
On success its return is the envelope's `friction` key; a nil hook yields the empty string.

`StepEnvelope` takes the friction status as a new `friction string` parameter after `statusFile` and reports it as `friction`.
Its doc comment lists the key and stops pinning the key count in prose ("exactly the documented keys below").
The error envelope's key set is unchanged.
`TestStepEnvelope_KeySetIsExactlySixteen` becomes `TestStepEnvelope_KeySetIsClosed` with `friction` in its want set, and the file's header comment stops naming a count.
`internal/loomcli/step_test.go`'s one `StepEnvelope` call passes the new argument.

New `step_test.go` cases:

- `AfterStep` runs after `PostStep` and before the envelope is printed, receives the step's result and a nil error, and its return is the envelope's `friction` value;
- `AfterStep` runs on a producer error and on a busy error, receives that error, and the error envelope carries no `friction` key;
- `AfterStep` does not run when `PreStep` or `BuildShed` refuses;
- a nil `AfterStep` leaves `friction` empty on a success envelope.

`CONSTRAINTS.md`'s Shed Verb-Set clause on the step envelope adds that the success envelope also carries `friction`.

**ImpactSummary:** one new nil-by-default hook and one new envelope key in `internal/shedverbs`; lifecycle and batten leave the hook nil and only gain an empty `friction` key, and the only other `StepEnvelope` caller is one loomcli test.
