# Card 26 — drift-retire-skip-keys

**Edit:**
- `internal/driftengine#Config`
- `internal/driftengine#configFile`
- `internal/driftengine#ConfigSpec`
- `internal/driftengine#LoadConfig`
- `internal/driftengine#checkScope`
- `internal/driftengine/template.yaml`
- `internal/driftengine#CaptureRequest`
- `internal/driftengine#CompareRequest`
- `internal/driftengine#RunCapture`
- `plan:internal/driftengine#CaptureCurrent`
- `internal/driftengine#RunCompare`
- `internal/driftengine#renderCommand`
- `internal/driftengine#Paths`
- `internal/driftengine#NewPaths`
- `internal/driftengine#noFileReason` -> `func noFileReason(baselineDir string) string`
- `internal/driftengine#awaitWaysForward` -> `func awaitWaysForward(onBranchPoint bool) string`
- `internal/driftengine#FailedCaptureError`
- `internal/driftengine#DriftAwait.await`
- `internal/driftengine#DriftBaseline.captureAgain`
- `internal/driftengine#TestLoadConfig`
- `internal/driftengine#templateConfig`
- `internal/driftengine#TestSeatTables`
- `internal/driftengine#TestRunCapture`
- `internal/driftengine#TestRunCompare`
- `plan:internal/driftengine#TestCaptureCurrent`
- `internal/driftengine#TestFailedCaptureError`
- `internal/driftengine#TestDriftAwait`
- `internal/driftengine#TestDriftBaseline`
- `internal/driftengine#baselineFixture`
- `internal/driftengine#newBaselineFixture`
- `internal/driftengine/doc.go`
- `internal/loomcli#driftDeps`
- `internal/driftcli#runCaptureChild`
- `plan:internal/driftcli#worktree.checkDeps`
- `internal/driftcli_test#driftYAML`
- `internal/driftcli_test#TestDrift_RunsInTheAnchoredDirectory`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configcli#TestDispatchHubWideBoard`

**Delete:**
- `internal/driftengine#writeSkipFile`
- `internal/driftengine#emptyUnderSkips`
- `internal/driftengine#rescopeWaysForward`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/driftcli/doc.go`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Retire `skip` and `verify_max_runs` and the `{skip_file}` placeholder (D3, D14, D15).

- `template.yaml` drops both keys and `{skip_file}` from the `capture` and `compare` comments; `Config` and `configFile` drop `Skip` and `VerifyMaxRuns`; `LoadConfig` drops their checks; `checkScope` checks the scope alone.
- `ConfigSpec` moves both keys from `Categories` to `RetiredKeys` under `CategoryNone`, so a board entry whose `run_settings` still carries one seeds and the load drops the key, and `lym config reconcile` prunes them from a hub base (`PATTERN-config-layers`).
- `renderCommand` refuses a command still holding `{skip_file}`: `drift: the <site> command holds {skip_file}, which drift no longer renders; way forward: remove it from <step> in the hub's drift.yaml with lym config drift --set from the prime` (`PATTERN-refusal-way-forward`); the placeholder constant stays for that check.
  `RunCapture`, `CaptureCurrent` and `RunCompare` call it first, before the slot, the lock, the running record and the baseline backup, so a refused command takes no slot and moves no baseline; `CaptureCurrent` returns the refusal as `Passed` false with it as `Reason`, as card 9 returns a step refusal, so Drift-Verify's Stuck and the gate's findings carry the edit.
- `CaptureRequest` and `CompareRequest` drop `Skip`, `Paths` drops `SkipFile`, and `writeSkipFile` goes; `loomcli`'s `driftDeps`, `runCaptureChild` and `checkDeps` stop passing a skip list.
  The capture record's `skip` field stays readable for a capture taken before this lands.
- `noFileReason` no longer names skipped tests, and `awaitWaysForward` loses its rescope branch, whose ways forward named the removed `lym drift scope`; `emptyUnderSkips` and `rescopeWaysForward` go, and `FailedCaptureError`, `DriftAwait.await` and `DriftBaseline.captureAgain` call the narrowed forms.

Tests: `TestLoadConfig` drops the skip and `verify_max_runs` rows and gains one where `skip` and `verify_max_runs` in a local layer load with the keys dropped; `templateConfig` and `TestSeatTables` drop the `VerifyMaxRuns` they set; `TestRunCapture` and `TestRunCompare` gain a row where a command holding `{skip_file}` is refused naming the edit, with the slot free and the baseline untouched after it, and drop the skip-file rows; `TestCaptureCurrent` gains a row where such a command is `Passed` false with the refusal as its reason; `TestFailedCaptureError`, `TestDriftAwait` and `TestDriftBaseline` drop their rescope rows; `newBaselineFixture` stops setting `Config.Skip`, so `baselineFixture`'s `configSkip` field and the `TestDriftBaseline` row that sets it go; `TestPrepareRunSettings` gains a row where `{"drift": {"skip": ["t"], "verify_max_runs": 3}}` seeds; `TestModules_EveryTemplateKeyClassified`'s drift local keys drop both; `TestDispatchHubWideBoard`'s drift rows change `judge_model` where they changed `verify_max_runs`; the driftcli fixtures stop seeding `verify_max_runs`.

**ImpactSummary:** Removes two keys and a placeholder from drift's config and runners; three wiring sites drop the skip list; a hub command still holding `{skip_file}` is now refused at run time with the edit named, as D15's notice warned.
