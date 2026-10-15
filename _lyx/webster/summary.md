# Engine model choice: every role's spawn spec carries its alias's engine

All 13 batches reported `status: done` on the first `record-batch`, with no recovery and no friction note.
The batch list had no batch 06, because card 6 (round-usage-engine) ran inside batch 05 with card 5.
Deviations below are the scope guard's lists of files changed outside a batch's own targets; they are informational.

- **01 spec-embeds-model-choice** (`88478912e`): `Spec` and `OpenSpec` embed `ModelChoice`, `Result` carries the run's engine, and `lyx shuttle open` opens on the alias's engine. Deviation: `internal/shuttleengine/spec_test.go`.
- **02 seat-model-choice** (`04d4c3ca6`): seats carry a `ModelChoice`, and every seat resolver builds it with `NewModelChoice`. No deviations.
- **03 burler-model-choice** (`96f9a05ac`): burler's per-round lists, `RunOpts` and the Bouncer pair became shuttle `ModelChoice`s, and `burlerengine.ModelChoice` was deleted. No deviations.
- **04 role-settings-model-choice** (`a1e413981`): driver, arbiter, alignment and tackler settings carry a `ModelChoice`, and `DriverSettings` was deleted. No deviations.
- **05 treadle-model-choice**, with card 6 (`7765e3114`): treadle's profile carries judge and attempt `ModelChoice`s, and `round-N-usage.yaml` records each half's and Bouncer pass's engine. Deviation: `internal/shedadapters/burlerreview_test.go`.
- **07 webster-roles-record-engines** (`fab2b84a2`): webster roles resolve to `ModelChoice`s, and `state.json` records each session's engine. Deviation: `internal/websterengine/state_test.go`.
- **08 webster-session-engines** (`b8bef9205`): webster reads each session through its recorded engine via a told lookup. Deviations: `internal/testkit/shedfake/shedfake.go`, `internal/websterengine/auditengine.go`, `auditengine_test.go`, `contractevidence_test.go` and `recoverbatch_test.go`.
- **09 webster-spawn-engine-checks** (`2d31829a9`): `run` and `recover-batch` refuse a role engine without fork audit before any mutation. Deviations: `internal/websterengine/recoverbatch_test.go` and `runlevel_test.go`.
- **10 merriam-instruction-files** (`017c90960`): `MerriamBase` counts the instruction files Master's engine names. Deviations: `internal/webstercli/cli.go`, `cli_test.go`, `reset_test.go`, and `internal/websterengine/merriambase_test.go`, `runlevel.go` and `runlevel_test.go`.
- **11 start-requirements-model-key** (`3ee9964b0`): shuttle's Start requirements refusal became a typed error whose generic way forward a caller's `ModelKey` replaces. Deviation: `internal/shuttleengine/requires.go`.
- **12 seat-requires** (`601e992a3`): seats declare their requirements, and a Start refusal names the model key. Deviations: `internal/loomengine/config.go` and `internal/seatengine/seatfake_test.go`.
- **13 master-driver-requires** (`d8d4307dc`): Master and the loom driver declare `background-wake`. No deviations.
- **14 model-choice-mapping-scan** (`69464e9d6`): `PATTERN-model-choice-mapping` and its scan test. Deviation: `internal/shuttleengine/modelchoicemapping_enforcement_test.go`.

Scope guard result: every deviation is a test file, a doc-adjacent helper or a file the batch's fork touched to keep the build green; none was treated as a failure.
