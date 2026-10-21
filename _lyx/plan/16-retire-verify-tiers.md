# Card 16 — retire-verify-tiers

**Edit:**
- `internal/languageconfig#config`
- `internal/languageconfig#Language`
- `internal/languageconfig#ConfigSpec`
- `internal/languageconfig#Load`
- `internal/languageconfig#VerifyTiersValue`
- `internal/languageconfig_test#TestLoad_ResolvesSkills`
- `internal/languageconfig_test#TestLoad_InvalidFailsNamingKey`
- `internal/languageconfig_test#TestVerifyTiersValue`
- `internal/languageconfig/template.yaml`
- `internal/languageconfig/doc.go`
- `internal/loomengine#PlanTable` -> `func PlanTable(layout *lyxcwd.Location, dirs configengine.Dirs, stencilsDir, specsDir string, cfg Config, reg modelspec.Registry, gateTiers string) (seatengine.Table, error)`
- `internal/loomengine#TestPlanTable`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/shedadapters#RubricNotes`
- `internal/shedadapters#ReadRubric`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.Call`
- `internal/shedadapters#TestReadRubric`
- `internal/shedadapters#TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#TestBurlerReviewEntry_RubricStencil`
- `internal/shedrecipe#TestBouncerEntry_HappyPath`
- `internal/testkit/envkit#FullEnv`
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir, gateTiers string, renameTools []string, outcomePath, summaryPath string, selfFixCap int, notePath, packPath string, layout PackLayout, pack []planindex.CardPack) (prompt, packFile []byte, err error)`
- `internal/websterengine#RenderRecoveryPrompt` -> `func RenderRecoveryPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, failureDigest, uncommittedPaths, reportPath, repoRoot, planDir, promptWorktreeRoot, stencilsDir, specsDir, gateTiers string, renameTools, patternChanges []string, selfFixCap int, notePath, parentName string) ([]byte, error)`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#BeginDeps`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/websterengine_test#TestTemplates_FillRequiresEveryRequiredMarker`
- `internal/websterengine_test#testVerifyTiers`
- `internal/websterengine_test#renderForkWithPack`
- `internal/websterengine_test#renderRecoveryWithUncommitted`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestRenderPrompts_ParentDirective`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine_test#TestRecoverBatch_UncommittedPathsPrompt`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#websterCLI.beginDeps`
- `internal/webstercli#websterCLI.recoverDeps`
- `internal/webstercli#TestWireHub_LanguageSkills`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_LanguageSkills`
- `contracts/stencils#rubricMarkerAllowlist`
- `contracts/stencils#TestLoomRubrics`
- `contracts/stencils#TestHubGenericStencils_NameNoGoTiers`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/specs/loom-plan-spec.md`

**Intent:**
Retire `language.yaml`'s `verify_tiers` (D9): every prompt that rendered it renders the hub's tiers and gate lists from `gate.yaml` instead, through `gateslot.Config.TiersValue()` (card 2), one line per tier with its number and `about` and one line per Go-read gate's list.

`verify_tiers` becomes a retired key of the language module (`CategoryGuard`), dropped at load and pruned by reconcile; `Language.VerifyTiers`, its template line, its validation and its doc paragraphs go, and the doc's Go-hub example moves to gate.yaml's tiers.
`VerifyTiersValue` keeps rendering `rename_tools`, its doc saying so.
The marker is renamed `gate_tiers` in `loom-template-plan.md`, `loom-rubric-plan-review.md` and `webster-body-implementer.md`, and every value that filled it becomes the rendered string:
`PlanTable`'s parameter, `RubricNotes`' and `BouncerConfig`'s field, `shedrecipe.Env`'s field, `BeginDeps`' and `RecoverDeps`' field, and the two render functions' parameter.
`loomcli`'s wiring and `webstercli`'s hub and standalone wiring load `gate.yaml` through `gateslot.LoadConfig` once and pass `TiersValue()`; a load failure there refuses the verb naming `lyx config gate`, as every other hub-config load at wiring does, and a standalone worktree with no `gate.yaml` renders `none`.

The plan's `## verify:` section is no gate input (D9), so the writer and reviewer stop asking for it:
`loom-template-plan.md` drops the section from the overview's optional sections and replaces its verify paragraphs with one saying the gates run the hub's tier lists over scopes lyx computes, the hub's tiers and gate lists following in the `gate_tiers` marker, a card's `**Verify:**` the optional escape hatch for what no gate expresses, and a `**Verify:**` line that tests is one `lyx gate test --wait` call with a tier list and scope form;
`loom-rubric-plan-review.md`'s verify-coverage bullet becomes a bullet that a card's `**Verify:**` line naming a raw test command is a finding, with the hub's tiers following, and its `publish_verify` clause goes;
`loom-template-rework.md` stops carrying `## verify:` forward;
`loom-plan-spec.md`'s verify model section is rewritten around the gates and tier lists, the `## verify:` section is described as accepted and ignored with an informational finding (card 18), and its examples use `lyx gate test --wait` calls;
`webster-body-implementer.md`'s raw-run paragraphs name the `gate_tiers` marker.

Tests: `TestLoad_ResolvesSkills` and `TestLoad_InvalidFailsNamingKey` drop their `verify_tiers` rows and gain one loading a file that still sets it; `TestVerifyTiersValue` keeps its `rename_tools` rows.
`TestPlanTable`, `TestReadRubric`, `TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts`, `TestBurlerReviewEntry_RubricStencil`, `TestBouncerEntry_HappyPath`, the webster prompt tests and the two wiring tests assert the rendered tier lines reach the `gate_tiers` marker.
`TestWire_LanguageSkills` flips its `verify_tiers` rows, the empty-entry halt included, to a file still setting the key loading as retired, and gains a row where an unloadable `gate.yaml` refuses wire; `TestWireHub_LanguageSkills` gains the same refusal row, and a standalone wiring with no `gate.yaml` rendering `none`.
`rubricMarkerAllowlist` and `TestLoomRubrics` swap the marker name; `TestHubGenericStencils_NameNoGoTiers` keeps barring a Go tier command from the hub-generic stencils and now names `gate_tiers` as their route.

**ImpactSummary:** one language key retires and its seven threading sites carry a rendered string instead; three stencils and the plan spec drop the `## verify:` section's role; no hub file fails to load.
