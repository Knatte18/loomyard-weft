# Card 5 — burler reads lenses

**Edit:**
- `internal/burlerengine#Engine`
- `internal/burlerengine#New` -> `func New(shuttle seatengine.Shuttle, geom Geometry, cfg lenses.Config, stencilsDir, frictionDir string, languageSkills []string) *Engine`
- `internal/burlerengine#Profile`
- `internal/burlerengine#Profile.validateReview` -> `func (p *Profile) validateReview(worktreeRoot string, cfg lenses.Config) error`
- `internal/burlerengine#Profile.validateFix` -> `func (p *Profile) validateFix(worktreeRoot string, cfg lenses.Config) error`
- `internal/burlerengine#resolveRecordedFan` -> `func resolveRecordedFan(cfg lenses.Config, fan string) ([]lenses.Lens, error)`
- `internal/burlerengine#applyClusterExclude` -> `func applyClusterExclude(fan string, resolved []lenses.Lens, exclude, held []string) []lenses.Lens`
- `internal/burlerengine#dropLenses` -> `func dropLenses(resolved []lenses.Lens, names []string) ([]lenses.Lens, map[string]struct{})`
- `internal/burlerengine#lensList` -> `func lensList(resolved []lenses.Lens) string`
- `internal/burlerengine#lensNames` -> `func lensNames(resolved []lenses.Lens) []string`
- `internal/burlerengine#Engine.Review`
- `internal/burlerengine#Engine.ProbeReview`
- `internal/burlerengine#Engine.ResumeReview`
- `internal/burlerengine#Engine.Fix`
- `internal/burlerengine#Engine.ProbeFix`
- `internal/burlerengine#Engine.ResumeFix`
- `internal/burlerengine#Engine.concludeFix`
- `internal/burlerengine#clusterRulesBlock`
- `internal/burlerengine#selfCheckBlock`
- `internal/burlerengine/doc.go`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/burlercli/doc.go`
- `internal/burlercli/run.go`
- `internal/loomcli#loomCLI.wire`
- `docs/overview.md`
- `internal/burlerengine#newEngineWith` -> `func newEngineWith(t *testing.T, geom Geometry, cfg lenses.Config, shuttle *fakeShuttle) *Engine`
- `internal/burlerengine#newEngineForTest`
- `internal/burlerengine#clusterTestConfig`
- `internal/burlerengine#testClusterFanConfig` -> `func testClusterFanConfig() lenses.Config`
- `internal/burlerengine#TestEngine_Review_Outcomes`
- `internal/burlerengine#TestEngine_Review_TableShape`
- `internal/burlerengine#TestEngine_Fix_Outcomes`
- `internal/burlerengine#checkFixTableShapeAndFiles`
- `internal/burlerengine#TestProfileValidate_ClusterExclude`
- `internal/burlerengine#TestProfile_Validate`
- `internal/burlerengine#TestComposePrompt_ClusterRules`
- `internal/burlerengine_test#newClusterSmokeEngine`
- `internal/burlerengine_test#TestSmokeBurlerRoundToyFixture`
- `internal/burlercli#TestRunCmd`

**Uses:**
- `plan:internal/lenses#Config`
- `plan:internal/lenses#Lens`
- `plan:internal/lenses#Load`
- `plan:internal/lenses#ResolveFan`
- `plan:internal/lenses#MaxFanLenses`

**Intent:** burler's round logic consumes the lenses module's types and load, and holds no lens or fan data of its own (D1).
`Engine.cfg` and `New`'s `cfg` become `lenses.Config`; `Profile`'s resolved `clusterLenses` and `fixLenses` become `[]lenses.Lens`; `validateReview`, `validateFix`, `resolveRecordedFan`, `applyClusterExclude`, `dropLenses`, `lensList` and `lensNames` take the moved types, their `lenses` parameters and locals renamed so none shadows the package.
`resolveRecordedFan` resolves through `lenses.ResolveFan` and its way forward names `lenses.yaml`: `define the fan in lenses.yaml, then resume the run or re-run the verb`.
`burlerCLI.wireHub`, `burlerCLI.wireStandalone` and `loomCLI.wire` load through `lenses.Load` instead of `burlerengine.LoadConfig`, which degrades to the template on an absent file, so the comments there about a zero `Config` are replaced by that fact.
burlerengine's own `Config`, `LoadConfig`, `ResolveFan` and fan key grammar stay in place, unused by these members, until card 8 deletes them.
The listed callers of the re-signed members change only as the new types force; every test that built a `Config` or `[]Lens` builds the `lenses` one, and `testClusterFanConfig`'s over-cap fan is `lenses.MaxFanLenses+1` long.
`New`'s doc comment names `cfg` as the `lenses.yaml` lens and fan library loaded via `lenses.Load`.
`profile_test`'s way-forward assertion in `TestProfile_Validate` names `lenses.yaml`.
`burlerengine/doc.go`'s cluster fan-out section, `burlercli/doc.go` and `run.go`'s profile template comments name `lenses.yaml` and the `lenses` package as the home of lenses and fans; the doc drops "seed-only" and the per-name template fallback, says a fan resolves against the loaded `lenses.yaml`, whose load fills a missing template lens or fan, and names the cap `lenses.MaxFanLenses` where it named `maxClusterN`.
`docs/overview.md`'s burler paragraphs, the `review-code` and `fix-code` fans sentence and the `cluster-fan` sentence, name `lenses.yaml` and drop "seed-only", in this card since the behaviour they describe changes here.
Test comments the switch makes wrong change with it: `testClusterFanConfig`'s doc comment names `lenses.ResolveFan` and `lenses.MaxFanLenses`;
`smoke_cluster_test.go`'s header comment names a `lenses.Config` built in Go with no `lenses.yaml` seeding, and `newClusterSmokeEngine`'s doc comment cites `lenses.Load` and `lenses.ResolveFan` where it cited `config.go`'s loader;
`burlercli`'s `wiring_test.go` header comment says the lenses load degrades to its template, and `TestRunCmd`'s comment on `newRound` names an empty `lenses.Config`.

**ImpactSummary:** burlerengine, burlercli and loomcli switch from burlerengine's config to internal/lenses; New's three production callers and its test callers are listed, and runtime fan resolution gains only lenses.yaml's per-name template fill, which gives the same lenses as today's fallback.
