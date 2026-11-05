# Card 18 — drift-baseline-await

**Edit:**
- `internal/driftengine#DriftBaseline.Call`
- `internal/driftengine#DriftBaseline.drive`
- `internal/driftengine#DriftBaseline.captureAgain`
- `internal/driftengine#DriftAwait.await`
- `internal/driftengine#Config`
- `internal/driftengine#LoadConfig`
- `internal/driftengine#checkScope`
- `internal/driftengine#TestDriftBaseline`
- `internal/driftengine#baselineFixture`
- `internal/driftengine#newBaselineFixture`
- `internal/driftengine#checkRecordedScope`
- `internal/driftengine#TestDriftAwait`
- `internal/driftengine#TestLoadConfig`
- `internal/driftengine/doc.go`
- `internal/shedrecipe#driftBaselineEntry`
- `internal/shedrecipe#TestDriftEntries`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/runprofile/template.yaml`
- `internal/runprofile_test#TestTemplateDocsProfile`

**Delete:**
- `internal/driftengine#settleConfiguredScope`
- `internal/driftengine#configuredBaselineReport`

**Uses:**
- `plan:internal/driftengine#WriteRecords`
- `plan:internal/driftengine#BaselineDigest`
- `plan:internal/driftengine#BaselineTrailerKey`

**Intent:** Drift-Baseline becomes code only (D3, D4) and Drift-Await records the baseline digest (D7).

- `LoadConfig` admits a stored `scope: partial`, with or without `skip`, and reads it as `full`, setting `Config.ScopeWasPartial`, so a base not yet edited still loads; `checkScope` keeps refusing an unknown scope and a `skip` under `off` or `full`.
- `DriftBaseline.drive`, after ending any live chair of the removed roles `drift-baseline` and `drift-verify` through `EndRoles` (D4), records the scope once through `WriteRecords` when none is recorded: `off` when `drift.scope` is `off`, with `ScopeSource` `drift.scope`, and `full` otherwise, with the config's `diff` and `key_patterns` (D3).
  A config scope read from `partial` is recorded `full` with one `logger.Warn` naming the key.
  A recorded scope, `off`, `full` or an old `partial`, is kept, and a later config change changes neither it nor the recorded mode and patterns.
  It spawns no agent and writes no `baseline.md`; the capture then starts as today, unless the scope is `off`.
  Its capture Stucks name `lym drift defer` behind `DeferHaltPrefix`, as before behind `CaptureHaltPrefix`, which stays first.
- `driftBaselineEntry` no longer requires `Env.Seats` or `BaselineTable`, and requires `EndRoles`, `CommitWithTrailers`, `TrailerCommit` and `FileAt`.
- `DriftAwait.await` writes `awaited_at` through `WriteRecords` with the extra trailer `Drift-Baseline: <BaselineDigest of the baseline directory>`, so the digest lives in a weft commit made before the writer (D7); the capture log is still committed on its way out.
- The `docs` profile's `darn` block in `internal/runprofile/template.yaml` sets `drift: {scope: off}` (D3).
- The recipe headers and Drift-Baseline's comment say the row records the scope from config with no agent; the darn header's note on a run that passed Drift-Baseline before Drift-Await names `lym drift defer` at Drift-Verify in place of `lym drift waive` (D12).
  Both headers add that a run parked at a Drift-Verify halt from before this change goes on with `lym loom resume`, which checks it under the new Drift; the note names none of the verbs card 22 removes (D14).

`TestDriftBaseline` rows: `drift.scope: off` records `off` with its source and takes no capture; an unset scope records `full` with the config's `diff` and patterns; a config `partial` records `full` (the Warn is not asserted, since the logger's output is process-global); an existing record survives a config change; no agent is spawned; a live chair of either removed role is ended before the scope is recorded; each scope write is one commit carrying `Drift-Records`.
`TestDriftAwait` gains a row: the counted capture's commit carries `Drift-Baseline` equal to the baseline directory's digest.
`TestLoadConfig` gains rows: `partial` with and without `skip` loads as `full` with `ScopeWasPartial`.
`TestDriftEntries` drops Drift-Baseline's `Seats` and `BaselineTable` rows and adds `EndRoles`.
`TestTemplateDocsProfile`'s want for the docs profile's darn settings gains `"drift": {"scope": "off"}`, which pins D3's criterion.

**ImpactSummary:** Removes the baseline agent from the Drift-Baseline row and changes how two rows commit their scope writes; the seat builder and stencil stay until card 24; one profile block gains a drift key.
