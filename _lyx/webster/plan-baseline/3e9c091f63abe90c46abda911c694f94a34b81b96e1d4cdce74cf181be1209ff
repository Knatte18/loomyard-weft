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
A `shedverbs` test comment that cites a deleted `loomcli` test (for example `TestStepEnvelope_KeySetIsExactlySixteen`'s "mirrors internal/loomcli/step_test.go's own closure test") is rewritten to stand alone.
The loom-specific tests stay: `TestStepEnvelope_NextInterruptPolicyMatchesTable`, `TestStepKindForBootstrapStage`, `TestStepCmd_BusyRefusal_BeforeBootstrap`, `TestStatusCmd_EnvelopeKeySet`.
`validate-plan`'s tests in `loomcli` and `webstercli` are not touched (D14).
What this lets through, bounded per D14: `loomcli` no longer re-checks `shedverbs`' step envelope and status-line rendering itself, so a regression there is caught only by `shedverbs`' own tests;
every deleted case is first asserted there, and `loomcli`'s own wiring of `shedverbs` stays asserted by the loom-specific tests that stay.
Every deleted test and every moved case goes in the commit body for the count accounting.

**ImpactSummary:** Test-only; duplicate `loomcli` tests leave, their unique cases land in `shedverbs`, the Shed Verb-Set enforcer is unchanged.
