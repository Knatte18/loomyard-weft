# Card 2 — step-after-hook

**Edit:**
- `internal/shedverbs/spec.go`
- `internal/shedverbs/step.go`
- `internal/shedverbs/step_test.go`
- `internal/shedverbs/run.go`
- `internal/shedverbs/run_test.go`
- `internal/loomcli/step_test.go`
- `internal/battencli/arm.go`
- `CONSTRAINTS.md`

**Create:**
- `internal/battencli/postrun_test.go`

**Uses:**
- `internal/shedengine#StepResult`
- `internal/shedengine#ErrShedBusy`
- `internal/output#ErrFields`

**Intent:**
Give the generic `step` body the after-step extension point `run` already has in `PostRun`, and a `friction` key on every step envelope, so loom can write halt notes and reflect inside the step process before the envelope is written, and report the reflection's status on a `failed` halt's error envelope as on a `blocked` halt's success envelope (D2 and its acceptance criterion).

`Hooks` gains `AfterStep func(ctx context.Context, res shedengine.StepResult, stepErr error) string`.
`stepCmd` calls it once `shed.Step` has returned, on every path: on success after `PostStep`, and on every `shed.Step` error, busy included, before the refusal envelope.
It is not called when `PreStep` or `BuildShed` refuses, since no step ran.
Its doc mirrors `PostRun`'s: the busy error is passed through so the hook applies its own busy skip.
Its return is the envelope's `friction` key on the success envelope and on the error envelope of a `shed.Step` error alike; a nil hook yields the empty string.

`StepEnvelope` takes the friction status as a new `friction string` parameter after `statusFile` and reports it as `friction`.
Its doc comment lists the key and stops pinning the key count in prose ("exactly the documented keys below").
`stepErrFields` takes a `friction string` parameter and reports it as `friction` on every error envelope; the `PreStep` and `BuildShed` refusals pass the empty string, and `StepLocations`' doc, which names the keys every envelope carries, adds `friction`.
`TestStepEnvelope_KeySetIsExactlySixteen` becomes `TestStepEnvelope_KeySetIsClosed` with `friction` in its want set, and the file's header comment stops naming a count; the error-envelope key-set table in `step_test.go` adds `friction` to its want set.
`internal/loomcli/step_test.go`'s one `StepEnvelope` call passes the new argument.

The generic `run` body merges `PostRun`'s returned map onto the error envelope, through `output.ErrFields`, as it does onto the success envelope; `Hooks.PostRun`'s doc states the merge on both paths.
`battenPostRun` returns nil on a non-nil `runErr`, and its doc says so, so batten's error envelope keeps exactly today's keys; batten's recipe and teardown are untouched.
`TestRunCmd_PostRunRunsOnErrorPathBeforeEnvelope` keeps its ordering assertion and now asserts the hook's map does reach the error envelope.

What this lets through, and the bound: `AfterStep` is a new side-effect point on every `shed.Step` return, error paths included; it runs only after the step's producer call and lock release, never before a step or in place of one, and it can add exactly one key, `friction`, to either envelope.
On `run`, a module's `PostRun` extras now reach its error envelope; batten opts out, and loom's only extra is `friction`.

New `step_test.go` cases:

- `AfterStep` runs after `PostStep` and before the envelope is printed, receives the step's result and a nil error, and its return is the envelope's `friction` value;
- `AfterStep` runs on a producer error and on a busy error, receives that error, and its return is the error envelope's `friction` value;
- `AfterStep` does not run when `PreStep` or `BuildShed` refuses, and those error envelopes carry an empty `friction`;
- a nil `AfterStep` leaves `friction` empty on the success and error envelopes.

`postrun_test.go` asserts `battenPostRun` returns nil on a non-nil `runErr` even with `abandonedSession` set, and the `abandonedSession` map on a nil one.

`CONSTRAINTS.md`'s Shed Verb-Set clause on the step envelope adds that the success envelope and every error envelope also carry `friction`, and that `run`'s error envelope carries a module's `PostRun` extras.

**ImpactSummary:** one new nil-by-default hook and one new key on both step envelopes in `internal/shedverbs`, plus `run`'s error envelope gaining `PostRun` extras; `battencli` leaves the hook nil, so its step envelopes only gain an empty `friction` key, and opts out of error-path extras in its own `PostRun`; `shedcli` takes its `Spec` from `loomcli.ArmAt` or `battencli.ArmAt` and needs no change; the only other `StepEnvelope` caller is one loomcli test.
