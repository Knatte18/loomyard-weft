# Card 9 — step-envelope-yield

**Edit:**
- `internal/shedverbs#StepEnvelope`
- `internal/shedverbs#TestStepEnvelope_FieldMapping`
- `internal/shedverbs#TestLoop_RunsUntilStopAndPrintsOneEnvelope`

**Uses:**
- `plan:internal/shedengine#Yield`

**Intent:**
Carry a yield through `lyx shed step` and its `--until-stop` loop as a stop that is no halt (D4).
`StepEnvelope` derives `continue` as `res.State == StateRunning && res.Outcome != Yield`, so a yielded step prints `outcome: yield`, `state: running` and `continue: false`; its doc comment states the new derivation, and the key set stays closed.
The loop already ends on a success envelope whose `continue` is false, with stop `halted`; its envelope then carries the step's `outcome: yield` and the loop's `state: running` and `status_moved: false`, which the driver reads in the next card, and no failed stop is written.
Tests: `TestStepEnvelope_FieldMapping` gains a yield row (`continue` false, `outcome` `yield`, `state` `running`); `TestLoop_RunsUntilStopAndPrintsOneEnvelope` gains a row whose fake child writes a yield envelope: the loop runs one step, prints one envelope with `outcome` `yield` and leaves the status file running at the same row.

**ImpactSummary:** One derived key changes for the new outcome only; every other step envelope keeps its continue value.
