# Card 6 — caller-label-wiring

**Create:**
- `internal/loomcli#loomCLI.driverGoto`
- `internal/loomcli#loomCLI.driverStep`
- `internal/loomcli#TestGoDriverSeams_LabelShed`
- `internal/battencli#battenCLI.driverGoto`
- `internal/battencli#battenCLI.driverStep`
- `internal/battencli#childGotoArgv`

**Edit:**
- `internal/shedverbs#Spec`
- `internal/shedverbs#RunGoto` -> `func RunGoto(ctx context.Context, spec *Spec, target, by string) (shedengine.Status, error)`
- `internal/shedverbs#gotoCmd`
- `internal/shedverbs#RunUntilStop` -> `func RunUntilStop(ctx context.Context, spec *Spec, argv []string, attempt, by string, out io.Writer) int`
- `internal/shedverbs#stepCmd`
- `internal/shedverbs#TestGotoCmd_MovesBlockedRunOntoRow`
- `internal/shedverbs#TestGotoCmd_RefusesMissingOrUnknownTarget`
- `internal/shedverbs#TestLoop_ArbiterAttemptConsumedByTheWaiter`
- `internal/shedverbs#TestLoop_RefusalsBeforeAnyStep`
- `internal/shedverbs#TestStepCmd_BusyTreatment`
- `internal/shedverbs#TestStepCmd_SuccessEnvelopeCarriesIdentityAndProgress`
- `internal/shedverbs/doc.go`
- `internal/loomcli#loomCLI`
- `internal/loomcli#loomCLI.armAt`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#TestSpecFor_ScratchAndFrictionDir`
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.armAt`
- `internal/battencli#battenCLI.specFor`
- `internal/battencli#battenCLI.driverDepsFor`
- `internal/battencli#childGoto`
- `internal/battencli#TestSpecFor_FillsRunIdentityForSlug`
- `internal/battencli#TestBattenDriverDeps`
- `contracts/specs/refusal-spec.md`

**Intent:**
Fill the caller label card 5 records: the goto verb and the step verb that resumes a halted run pass it down, classified in the CLI layer from the strand variable through `agentname.CallerLabel` (card 4), and the Go driver and batten's automatic goto pass `shedengine.CallerShed`.
`shedverbs` gains no import, as `PATTERN-shed-verb-set` requires: it reads no environment and takes the label as a string.

The arming modules.
`Spec` gains `Caller string`, the label of the process the verbs run in; the empty string records none.
`loomCLI` and `battenCLI` gain a `caller` field, which their `armAt` fills with `agentname.CallerLabel(os.Getenv(agentname.StrandNameEnv))`, and their `specFor` copies into `Spec.Caller`, so `specFor` keeps doing no I/O of its own; this covers `lyx shed`'s verbs too, since `shedcli` arms through these two.
`TestSpecFor_ScratchAndFrictionDir` and `TestSpecFor_FillsRunIdentityForSlug` each set the receiver's `caller` to `orch` and assert `Spec.Caller` is `orch`.

Goto.
`RunGoto` takes `by` and passes it as `GotoRequest.By`.
`gotoCmd` gains a hidden `--by` flag that accepts `shedengine.CallerShed` alone, the label of lyx's own automatic steps, and passes the flag's value when set and `spec.Caller` otherwise.
Any other `--by` value is refused before the status file is read, with "goto --by takes only shed, the label of lyx's own automatic steps; way forward: re-run goto without --by", and the status file is left untouched.
Bound: `--by` can write no label but `shed`, and the label enters no Go decision, so the flag can only mislabel a report row, never skip or admit a goto.
`contracts/specs/refusal-spec.md` gains the row `goto caller label refused` beside the other goto rows, naming that way forward.

Step.
`RunUntilStop` takes `by`; after its refusals and the attempt's consumption pass and before the loop is spawned, it calls `shedengine.RecordResumer(spec.StatusPath, spec.StatusLockPath, by)`, logging a failure at Warn and spawning the loop anyway, since the label is for reports only.
`stepCmd` passes `spec.Caller` on `--until-stop`, and its plain step sets the built shed's `ResumedBy` to `spec.Caller` before `shed.Step`, so the label is recorded under the run lock by the resume write and a step refused as busy, or by `PreStep` or `BuildShed`, records nothing.
The first-label-stands rule of card 5 keeps a waiter's `shed` against the loop children's own labels.

The Go driver and batten.
`loomCLI.driverGoto` and `loomCLI.driverStep` hold the bodies of `goDriverDepsFor`'s `Goto` and `Step` seams, which now call them: `RunGoto` and `RunUntilStop` with `shedengine.CallerShed`.
`battenCLI.driverGoto` and `battenCLI.driverStep` do the same for `battenCLI.driverDepsFor`'s seams, `driverStep` keeping its `untilStop` override.
`childGotoArgv` returns the argument list `childGoto` spawns, `loom goto --to <target> --by shed`, so batten's automatic Publish verify retry labels its goto `shed`.
Batten's other automatic steps resume the child through its bootstrap, whose Go driver labels the resume `shed` through its `Step` seam.

Tests:
`TestGotoCmd_MovesBlockedRunOntoRow` runs with `spec.Caller` `orch` and asserts the goto entry's `By` is `orch`, and gains a row run with `--by shed` asserting `By` `shed`.
`TestGotoCmd_RefusesMissingOrUnknownTarget` gains a row `--to B --by orch` asserting exit 1, the error naming `--by` and `way forward: re-run goto without --by`, and the status file unchanged.
`TestLoop_ArbiterAttemptConsumedByTheWaiter` seeds the status blocked with `fx.setStatus` before the waiter runs, sets `spec.Caller` to `orch`, and its `Loop.Spawn` seam asserts the status's `PendingBy` is `orch` at spawn time, so the waiter records before any loop or child starts; a second subtest seeds a status file that does not decode and asserts the loop is still spawned.
`TestLoop_RefusalsBeforeAnyStep` asserts, on each of its refusal rows run from a blocked status with `spec.Caller` `orch`, that `PendingBy` stays empty, so no refused waiter records a label.
`TestStepCmd_SuccessEnvelopeCarriesIdentityAndProgress` gains a subtest seeding the status blocked with `spec.Caller` `orch`, asserting the appended entry's `By` is `orch` and `PendingBy` empty, and one with an empty `spec.Caller` asserting no `by`.
`TestStepCmd_BusyTreatment` seeds the status blocked with `spec.Caller` `orch` and asserts `PendingBy` stays empty after the busy refusal.
`TestGoDriverSeams_LabelShed` in `internal/loomcli/drivewiring_test.go` builds a `loomCLI` over a temp status file seeded blocked: `driverGoto` writes a goto entry with `By` `shed`, and `driverStep`, over a spec whose `Loop.Spawn` seam records the call and returns an error so no loop runs, leaves `PendingBy` `shed`.
`TestBattenDriverDeps` in `internal/battencli/driverdeps_test.go` gains the assertions that `childGotoArgv("Merge-In")` is `loom goto --to Merge-In --by shed`, and that the `deps.Goto` and `deps.Step` seams `driverDepsFor` returns, over the receiver's status seeded blocked with `untilStop` nil, write a goto entry with `By` `shed` and leave `PendingBy` `shed`.
`internal/battencli` keeps its row of 83 and `internal/loomcli` stays within its row, so the budget file is not touched; `internal/shedverbs` gains no top-level test.
`internal/shedverbs/doc.go` says the goto and step bodies record `Spec.Caller`, `RunGoto` and `RunUntilStop` take the label explicitly, and `--by` admits `shed` alone.

**ImpactSummary:** RunGoto and RunUntilStop each gain one parameter, their callers moving into the four new driver methods and the verb bodies on this card; Spec gains one field filled from the two arming modules' armAt; the only new refusal is the hidden flag's.

**Commit:** 6: Pass the caller label from the goto and step verbs and the Go driver
