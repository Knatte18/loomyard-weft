# Card 27 — shedverbs-duplicates

**Edit:**
- `internal/loomcli/step_test.go`
- `internal/loomcli/status_test.go`
- `internal/shedverbs/step_test.go`
- `internal/shedverbs/status_test.go`

**Intent:** Delete `loomcli`'s tests of `shedverbs` behaviour that `shedverbs` tests itself (D14, #327 §6), case by case.
First diff the two 16-key step-envelope lists once and record the result in the commit body.
Then delete `loomcli`'s `TestStepEnvelope_KeySetIsExactlySixteen`, `TestStepKinds_ClosedSetOfFive`, `TestStepEnvelope_ContinueTracksRunningState`, `TestRenderStatusLine` and `TestPrintStatusLinesOnChange`.
Before each deletion, every case it holds with no counterpart in `shedverbs`' own tests moves there first: at least `PrintStatusLinesOnChange`'s deduped unavailable line, the reprint after returning to an earlier line and the always-printed first line.
`TestStepEnvelope_FieldMapping` moves to `shedverbs/step_test.go` if `shedverbs` lacks an equivalent, else it is deleted as a duplicate.
The loom-specific tests stay: `TestStepEnvelope_NextInterruptPolicyMatchesTable`, `TestStepKindForBootstrapStage`, `TestStepCmd_BusyRefusal_BeforeBootstrap`, `TestStatusCmd_EnvelopeKeySet`.
`validate-plan`'s tests in `loomcli` and `webstercli` are not touched (D14).
Every deleted test and every moved case goes in the commit body for the count accounting.

**ImpactSummary:** Test-only; duplicate `loomcli` tests leave, their unique cases land in `shedverbs`, the Shed Verb-Set enforcer is unchanged.
