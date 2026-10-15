# Card 7 — burler-seat-skills

**Create:**
- `plan:internal/burlerengine#Engine.seatSkills` -> `func (e *Engine) seatSkills(p *Profile) []string`

**Edit:**
- `internal/burlerengine#Profile`
- `internal/burlerengine#Engine`
- `internal/burlerengine#New` -> `func New(shuttle seatengine.Shuttle, geom Geometry, cfg Config, stencilsDir, frictionDir string, languageSkills []string) *Engine`
- `internal/burlerengine#Profile.reviewTable` -> `func (p *Profile) reviewTable(opts RunOpts, values map[string]string, skills []string) seatengine.Table`
- `internal/burlerengine#Profile.fixTable` -> `func (p *Profile) fixTable(opts RunOpts, values map[string]string, skills []string) seatengine.Table`
- `internal/burlerengine#Engine.Review`
- `internal/burlerengine#Engine.ProbeReview`
- `internal/burlerengine#Engine.ResumeReview`
- `internal/burlerengine#Engine.Fix`
- `internal/burlerengine#Engine.ProbeFix`
- `internal/burlerengine#Engine.ResumeFix`
- `internal/burlerengine#reviewHalf`
- `internal/burlerengine#fixHalf`
- `internal/burlerengine#TestComposePrompt_MinimalProfile`
- `internal/burlerengine#TestComposePrompt_ParentDirective`
- `internal/burlerengine#TestComposePrompt_ClusterRules`
- `internal/burlerengine#newEngineWith`
- `internal/burlerengine#TestEngine_Review_TableShape`
- `internal/burlerengine#TestEngine_Fix_TableShapeAndFiles`
- `internal/burlerengine#TestEngine_ProbeReview_And_ResumeReview`
- `internal/burlerengine#TestEngine_ProbeFix_And_ResumeFix`
- `internal/burlerengine_test#newClusterSmokeEngine`
- `internal/burlerengine_test#TestSmokeBurlerRoundToyFixture`
- `internal/burlerengine/doc.go`
- `internal/loomcli#loomCLI.wire`
- `internal/burlercli#burlerCLI`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/burlercli#TestRunCmd`
- `internal/burlercli#TestWire_SelectsModeAndWiresItsFields`

**Uses:**
- `plan:internal/languageconfig#AppendSkills`
- `plan:internal/languageconfig#Load`

**Intent:**
A Burler half whose profile marks a code round loads the told language list after `burlerSkills` on every seat; any other half loads `burlerSkills` alone (D5).

`Profile` gains the exported field `LanguageSkills bool`, documented as marking a code round: when true, every seat of the half loads `burlerSkills` followed by the engine's language skills; when false, `burlerSkills` alone.
It only adds skills to a seat's load turn and changes no rubric, lens, fan or gate.
`Engine` gains the field `languageSkills []string`, and `New` gains the trailing parameter `languageSkills []string`, documented as the hub's language skills a code round's seats load after `burlerSkills`, empty for none.
`seatSkills` returns `burlerSkills` when `p.LanguageSkills` is false and `languageconfig.AppendSkills(burlerSkills, e.languageSkills)` when it is true.
`reviewTable` and `fixTable` gain the parameter `skills []string` and set their seat's `Skills` from it in place of `burlerSkills`;
`Engine.Review`, `Engine.ProbeReview` and `Engine.ResumeReview` pass `e.seatSkills(&p)` to `reviewTable`, and `Engine.Fix`, `Engine.ProbeFix` and `Engine.ResumeFix` pass it to `fixTable`, so a probe or resume builds the same table its spawn did.
The prompt-test callers `reviewHalf`, `fixHalf`, `TestComposePrompt_MinimalProfile`, `TestComposePrompt_ParentDirective` and `TestComposePrompt_ClusterRules` pass `burlerSkills`, since they read prompts and not skills.

`New`'s callers: `loomCLI.wire` passes `c.languageSkills`, the field card 5 loads and `TestWire_LanguageSkills` asserts;
`burlerCLI` gains the field `languageSkills []string`, and `burlerCLI.wireHub` calls `languageconfig.Load(fabricengine.ConfigDirs(loc))` beside its `burler.yaml` load, returns its error unchanged, stores the list in `c.languageSkills` and passes that field to `New`, so `lyx burler run` honours a profile's `language-skills` in a hub;
`burlerCLI.wireStandalone` leaves `c.languageSkills` empty, passes it, and loads no `language` module (D3);
`newEngineWith`, `TestRunCmd`, `newClusterSmokeEngine` and `TestSmokeBurlerRoundToyFixture` pass `nil`.
`internal/burlerengine/doc.go`'s skills paragraph adds the code-round rule and that the list is told to `New`.

`TestEngine_Review_TableShape` builds its engine with `New` directly (geometry shortname `tst`, slug `slug`, `clusterTestConfig`, `newTestStencilsDir(t)`) over a per-row language list and `Profile.LanguageSkills`, with a per-row `wantSkills`:
the existing rows tell no list and want `burlerSkills`' three skills;
a new row "code round" tells `scribe:golang-testing`, `scribe:testing` with `LanguageSkills` true and wants `scribe:prose`, `scribe:code-quality`, `scribe:testing`, `scribe:golang-testing`;
a new row "non-code round with a language list" tells the same list with `LanguageSkills` false and wants the three alone.
`TestEngine_Fix_TableShapeAndFiles` builds its engine with `New` directly over the language list `scribe:golang-testing`, sets `p.LanguageSkills` true, and asserts the fixer spec's `Skills` equal `scribe:prose`, `scribe:code-quality`, `scribe:testing`, `scribe:golang-testing`.
`TestEngine_ProbeReview_And_ResumeReview` and `TestEngine_ProbeFix_And_ResumeFix` (both in package `burlerengine`) set the engine's `languageSkills` to `scribe:golang-testing` and the profile's `LanguageSkills` true in their shared loop, and assert the probed spec `shuttle.probeSpecs[<half role>].Skills` equals `burlerSkills` followed by `scribe:golang-testing`, so a probe or resume that built its table without `seatSkills` fails them.
`TestWire_SelectsModeAndWiresItsFields` gains a "HubWithLanguageList" subtest that writes `<BoardDir>/_lyx/config/language.yaml` listing `scribe:golang-testing` under `fabricengine.BoardDir(hub)` and asserts `c.languageSkills` equals it, and a "HubWithInvalidLanguageList" subtest whose entry is empty and asserts `wire` fails naming `skills` and `lyx config language`;
its "Standalone" subtest additionally asserts `c.languageSkills` is empty.

**ImpactSummary:** `New` gains one parameter whose callers are in `burlerengine`, `burlercli` and `loomcli` (two of them `llm`-tagged smoke tests), the two unexported table builders gain one whose callers are all inside `burlerengine`, and `burlercli`'s hub wiring gains one hub-file load; a profile without the new field behaves exactly as today.
