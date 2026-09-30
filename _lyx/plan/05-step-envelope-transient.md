# Card 5 — step error envelope carries `transient`

**Edit:**
- `internal/shedverbs/step.go`
- `internal/shedverbs/step_test.go`

**Prosa:**
- `CONSTRAINTS.md`

**Uses:**
- `plan:internal/shedengine#TransientOf`
- `internal/shedverbs#StepKinds`

**Intent:**
Carry the mark to the driver (decision record D1): every error envelope `lyx shed step` prints gains a `transient` key holding the class name, or the empty string when the failure is not transient.
The refusal-kind vocabulary stays closed at five; `transient` is a new envelope key, not a kind.
The success envelope is unchanged.

In `stepCmd` (`step.go`), the refusal path takes the class alongside the kind:
- a `PreStep` error, a `BuildShed` error and a producer error report `string(shedengine.TransientOf(err))`;
- the nil-`BuildShed` refusal and the busy refusal report the empty string.
`stepErrFields` gains the `transient` parameter and key, and its doc comment, plus the file's own description of the error envelope, name the new key: kind, `transient`, and the five location keys.

`CONSTRAINTS.md`'s Shed Verb-Set Invariant bullet on the step envelope's key set adds that every error envelope also carries `transient`, and that the key is not a sixth kind.

Tests (`step_test.go`):
- `TestStepCmd_ErrorEnvelopesCarryLocations` asserts `transient` is present and empty on every row of its table, and asserts the exact error-envelope key set.
- A new table test drives a `PreStep` returning an error marked `TransientGitTransport` via `shedengine.MarkTransient`, a `BuildShed` returning one marked `TransientAgentStart`, and a producer row returning an error that the fake shed's `Transient` classifier marks `TransientGitHubAPI`.
  Each envelope carries its class; the kinds stay `unseeded`/`bootstrap`/`producer` as today.
- The busy refusal carries an empty `transient` even when the classifier would mark its error.
- `TestStepEnvelope_KeySetIsExactlySixteen` and `TestStepKinds_IsExactlyFive` stay as they are and keep passing.

**ImpactSummary:** Adds one key to every `lyx shed step` error envelope and one parameter to `stepErrFields`; consumers reading the envelope by key are unaffected, and the success envelope and the kind vocabulary are unchanged.

**Commit:** 5: shedverbs: step error envelopes carry the transient class
