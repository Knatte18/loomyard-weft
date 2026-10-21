# Card 15 — webster-prompt-gates

**Edit:**
- `internal/websterengine#cardGateCommand` -> `func cardGateCommand(card planparser.Card, gate gateslot.Config) ([]string, error)`
- `internal/websterengine#renderCardGates` -> `func renderCardGates(cards []planparser.Card, planDirDisplay string, gate gateslot.Config) (string, error)`
- `internal/websterengine#renderBatchGate` -> `func renderBatchGate(cards []planparser.Card, gate gateslot.Config) (string, error)`
- `internal/websterengine#RenderForkPrompt` -> `func RenderForkPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, reportPath, planDir, promptWorktreeRoot, stencilsDir, specsDir string, verifyTiers, renameTools []string, outcomePath, summaryPath string, selfFixCap int, notePath, packPath string, layout PackLayout, pack []planindex.CardPack) (prompt, packFile []byte, err error)`
- `internal/websterengine#RenderRecoveryPrompt` -> `func RenderRecoveryPrompt(batch batcher.Batch, cardGates, batchGate, prevDigest, failureDigest, uncommittedPaths, reportPath, repoRoot, planDir, promptWorktreeRoot, stencilsDir, specsDir string, verifyTiers, renameTools, patternChanges []string, selfFixCap int, notePath, parentName string) ([]byte, error)`
- `internal/websterengine#RenderVerifyFixPrompt` -> `func RenderVerifyFixPrompt(reportPath, worktreeRoot, planDir, stencilsDir, outcomePath, summaryPath, notePath, fixGate string) ([]byte, error)`
- `internal/websterengine#Run`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#BeginDeps`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#RecoverDeps`
- `internal/websterengine_test#renderVerifyFix`
- `internal/websterengine#TestCardGateCommand`
- `internal/websterengine#TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`
- `internal/websterengine#TestRenderBatchGate`
- `internal/websterengine#TestRenameBoundary_BatchesCheckAndGateTheirOwnTree`
- `internal/websterengine_test#testCardGates`
- `internal/websterengine_test#newBeginFixture`
- `internal/websterengine_test#newRecoverFixtureOver`
- `internal/websterengine_test#newRunFixtureOver`
- `internal/webstercli#newVerbsFixture`
- `internal/websterengine_test#forkTemplateMarkerValues`
- `internal/websterengine_test#TestTemplates_FillRequiresEveryRequiredMarker`
- `internal/websterengine_test#renderForkWithPack`
- `internal/websterengine_test#renderRecoveryWithUncommitted`
- `internal/websterengine_test#TestRenderPrompts`
- `internal/websterengine_test#TestRenderPrompts_ParentDirective`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine_test#TestRecoverBatch_SecondCall`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI.beginDeps`
- `internal/webstercli#websterCLI.recoverDeps`
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/webster/webster-prefix-recovery.md`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/webster/webster-body-verify-fix.md`
- `contracts/stencils/darn/darn-template-write.md`
- `contracts/specs/loom-plan-spec.md`
- `.gitattributes`

**Delete:**
- `internal/websterengine#planVerifyValue`
- `internal/websterengine#gateScope`
- `internal/websterengine#moduleScope`
- `internal/websterengine#gateScopeOf`
- `internal/websterengine#gateScope.steps`
- `internal/websterengine#packageArg`
- `internal/websterengine#guardTests`
- `internal/websterengine#scanGuardTests`
- `internal/websterengine#guardTests.step`
- `internal/websterengine#guardTests.holdsFilesAfter`
- `internal/websterengine#cardCreatedRefs`
- `internal/websterengine#guardNotice`
- `internal/websterengine#fillFragment`
- `internal/websterengine#skipGuardDir`
- `internal/websterengine#walkError`
- `internal/websterengine#writeGuardFixture`
- `internal/websterengine#guardSource`
- `internal/websterengine#guardSourceTwo`
- `internal/websterengine#misplacedGuardSrc`
- `internal/websterengine#TestScanGuardTests`
- `internal/websterengine_test#guardFixture`
- `internal/websterengine_test#guardSourceMarked`
- `internal/websterengine_test#guardSourceMisplaced`
- `internal/impactset#MaxCommandLength`
- `contracts/stencils#WebsterFragmentGuardScanError`
- `contracts/stencils#WebsterFragmentGuardTooLong`
- `contracts/stencils#WebsterFragmentGuardFix`
- `contracts/stencils/webster/webster-fragment-guard-scan-error.md`
- `contracts/stencils/webster/webster-fragment-guard-too-long.md`
- `contracts/stencils/webster/webster-fragment-guard-fix.md`

**Intent:**
The webster fork and recovery prompts render gate calls from the hub's gate lists, never a test command or a package list (D8, D9).

Each card's gate is `lyx gate test --wait --tier <card list> --card <id>`, rendered through `gateslot.Config.GateCall("card", "--card", <id>)`, followed by `lyx loom lint-comments` and `lyx loom lint-pattern` as today; the gate's own card scope computes the packages and runs the guard step (card 9), so the Go-side scope, the guard scan, its notices and their three fragment stencils go, and `impactset.MaxCommandLength` with its last readers.
The batch gate is one call, `GateCall("batch", "--card", <id>, "--card", <id>, ...)` over every card of the batch.
`BeginDeps` and `RecoverDeps` gain `Gate gateslot.Config`, which `webstercli` loads from the hub's `gate.yaml` at begin-batch and recover-batch through `gateslot.LoadConfig`, or the worktree's own file standalone; a `TierList` refusal refuses the verb before any mutation, with the refusal's way forward and role clause (card 2), since Merriam, not the prime, calls begin-batch.
The prompts stop rendering the plan's `## verify:` section, an in-flight plan's included: `RenderForkPrompt` and `RenderRecoveryPrompt` drop the `planVerify` parameter and the `plan_verify` marker.

`webster-body-implementer.md` is reworded:
step 3 runs each rendered gate step as today and describes the guard step as the gate call's own;
step 5 runs `lyx gate card-verify <card id>` once per card that declares a `**Verify:**` line, in place of running each line as a Bash call, and the foreground rule at the section head and the `FAILED` wording speak of that one call;
the slot-busy section keeps the exit-75 rule for a call made without `--wait`;
the raw-run paragraph names `lyx gate test` with a gate's tier list and a scope form as the one test route, never a raw test command;
the paragraph after the self-fix test that read the plan's verify command now says the `final` gate runs at the plan level and catches what the card gates do not.
`webster-prefix-recovery.md` and `webster-template-master.md` stop naming `## verify:` as a section to read, and the master template says the `final` gate runs at HEAD after its turn;
`webster-body-verify-fix.md` gains a required `fix_gate` marker, filled by `RenderVerifyFixPrompt` with `GateCall("round")`, the `round` list over the computed scope, which `Run` renders from the told geometry's `GateConfig` (card 12); the fixer runs that call before it reports, in place of `lyx gate test --tags integration <the findings' packages>`.
`darn-template-write.md`'s test step names `lyx gate test --wait` with its computed scope.
`loom-plan-spec.md`'s card-gate and batch-gate paragraphs describe the rendered calls.

Tests: `TestCardGateCommand` is cut to two rows: a card's gate call followed by its two lints, and a dangling `card` list refusing; its nested-module and delete-only rows go, since every card now renders the same `--card` call and `TestCardPackages` (card 6) covers their packages.
`TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`, `TestRenderBatchGate` and `TestRenameBoundary_BatchesCheckAndGateTheirOwnTree` assert the rendered calls; `TestScanGuardTests` and its fixtures go with the scan.
`TestBeginBatch_PromptFile` gains a row where a dangling `batch` list refuses begin-batch with no prompt file written and no state recorded.
It drops its guard rows (the marked-test row, the two misplaced-marker rows and the over-the-command-limit row that names `impactset.MaxCommandLength`) and recasts its remaining rows that expect the per-package `lyx gate test ./internal/alpha` step as the rendered `--card` call; `TestRecoverSpawnOrAttach` drops its row "a marked guard test renders the same guard step the begin prompt carries".
`newBeginFixture` and `newRecoverFixtureOver`, which `newRecoverFixture` delegates to, set the `Gate` field of `BeginDeps` and `RecoverDeps` to a `gateslot.Config` declaring `tier0` and the `card` and `batch` lists, `newRunFixtureOver` tells `Geom.GateConfig` a reader over a config declaring `tier0` and a `round` list, and `newVerbsFixture` seeds `<hub>/_board/_lyx/config/gate.yaml` the same way, since `hubgeom.WebsterGeometry`'s `GateConfig` (card 12) reads it; a zero config is refused by `TierList`, so every test built on these fixtures must reach the rendered gate calls instead of that refusal.
The prompt tests drop the `plan_verify` marker and assert the rendered gate calls and the `card-verify` step, `forkTemplateMarkerValues` stops filling `plan_verify`, and `TestTemplates_FillRequiresEveryRequiredMarker` stops requiring it;
`.gitattributes` drops the three fragment stencils' lines; `TestStencils_TopLevelMarkers` drops the three fragments and the `plan_verify` marker and gains the verify-fix stencil's `fix_gate`; `renderVerifyFix` passes a rendered call and the prompt test asserts it.

**ImpactSummary:** webster's card and batch gate rendering, its guard scan and the plan-verify marker are replaced by rendered gate calls; two render functions lose a parameter with their four callers, and three fragment stencils go.
