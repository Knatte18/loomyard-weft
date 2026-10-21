# Lens review — test-tiers plan

Eight lens forks ran once over the plan fork0 wrote; every fork returned a report.
Card numbers below are the plan's numbers after the fix that moved the round-gate card from 13 to 10, so old cards 10, 11 and 12 are now 11, 12 and 13.

## affected-set

- affected-set, BLOCKING, card 15 drops `plan_verify` while only card 16 lists the template tests requiring it: fixed as proposed, card 15 lists `forkTemplateMarkerValues` and `TestTemplates_FillRequiresEveryRequiredMarker` and drops the marker there.
- affected-set, BLOCKING, card 14's `landing` kind breaks `TestMergeIn_PlanVerifyFailureRecord`: fixed as proposed, card 14 lists it and asserts kind `landing` and the tier.
- affected-set, BLOCKING, card 14's retired `PublishVerify` breaks `TestWire_DefaultConfig`, `hubLocation`, `hubPublishVerify`: fixed as proposed, card 14 lists all three and drops the constant.
- affected-set, BLOCKING, card 19's removal of `Limits.GoParallel` breaks `TestGlyphChain_CallerCoverageTyped`: fixed as proposed, card 19 lists it and swaps in `Tier0Parallel`.
- affected-set, BLOCKING, `go_parallel` and `tier_parallel` readers unlisted (`TestModules_EveryTemplateKeyClassified`, `TestDispatchLayers`, `TestConfigOverRealHub`, `TestGateTest_Scenario`'s machine row, hubreconcile's stale-build test): fixed as proposed for the first four (card 2 pins `tier_parallel`, card 19 lists the rest and recasts them onto `tier_parallel`); disputed for hubreconcile, premise false: `internal/hubreconcile/ensure_integration_test.go:301-316` seeds the machine file and asserts only that its bytes are unchanged after `Ensure`, which a retired machine key keeps true.
- affected-set, MEDIUM, `TestLandingDeps_EveryFieldPopulated` names `PublishVerifySkip`: fixed as proposed, card 14 lists it.
- affected-set, LOW, `.gitattributes` keeps the deleted fragment stencils: fixed as proposed, card 15 edits it.
- affected-set, LOW, `docs/overview.md:362` names `go_parallel`: fixed as proposed, card 19 lists the file.
- affected-set, LOW, `TestWire_LanguageSkills`' `verify_tiers` rows unstated: fixed as proposed, card 16 states the flip.

## guard-reach

- guard-reach, MEDIUM, no card owns the darn writer gate's empty tree-scope pass (merged with criteria-coverage 6 and branch-coverage 10): fixed as proposed, card 10 adds the branch in `NewVerifyGate`'s darn form with a round-scenario row, and card 14 names the landing sites' path and a Publish row.
- guard-reach, MEDIUM, card 8's outside-hub read cannot happen for a directory in no git worktree: fixed as proposed, card 8 refuses that case naming `lyx config gate` and moves the outside-hub row into a standalone git worktree.
- guard-reach, MEDIUM, `--card` under `none` may skip the id refusals: fixed as proposed, card 9 runs the plan read and its refusals first under either scoper, with an unknown-id row under `none`.
- guard-reach, LOW, `Geometry.GateConfig` set only in webstercli (merged with forced-edits 1, BLOCKING there): fixed as proposed, card 13 sets it in `hubgeom.WebsterGeometry` and `standalonegeom.WebsterGeometry` with their tests, dropping the webstercli wiring edits.
- guard-reach, LOW, multi-line `**Verify:**` branches unreachable: fixed as proposed, cards 11 and 18 read a value as one line, and the overview records why.
- guard-reach, LOW, `Claude.Prepare`'s no-board claim contradicts the loader: fixed as proposed, card 17 loads the worktree's own file outside a hub and reads an absent file as an empty list.
- guard-reach, LOW, a missing tier summary line leaves `Tier` 0, off the failing gate's list: fixed differently, the proposed change named no fix; card 14 writes the gate's highest tier instead, and card 10 reads an off-list tier the same way, so the confirm pass always runs a tier of the failing gate.
- guard-reach, LOW, the drain row implies a spawn in an untagged test (merged with row-reachability 8): fixed as proposed, card 3's drain holder is an in-process multi-slot waiter.

## way-forward

- way-forward, MEDIUM, refusals naming `lyx config gate` send forks to a prime-only step: fixed as proposed, card 2's refusals and card 8's carry `withConfigWayForward`'s role clause, cards 10, 13, 14 and 15 pass it through, and the overview records it.
- way-forward, MEDIUM, card 18 strands an in-flight rework plan with a done card: disputed, premise false: `internal/loomshed/gates.go:335` documents that `ValidateReworkPlan` checks the fresh generation, numbered from `first_card` (`internal/planparser/firstcard.go`), which holds no done card, and `ValidatePlan` passes only pending cards (`gates.go:261-271`); card 18 now states this.
- way-forward, MEDIUM, standalone `ErrConfigAbsent` names `lyx fabric reconcile`: fixed as proposed, card 2's `LoadConfig` names `lyx config gate` outside a hub, and cards 8, 13 and 17 inherit it.
- way-forward, LOW, no reason names raising `timeout_min`: fixed as proposed, card 4's timeout line and card 12's landing reason name it.
- way-forward, LOW, the pre-tier ceiling names no next step: fixed as proposed, card 8 names re-running the same command.
- way-forward, LOW, `card-verify`'s no-plan refusal names no next step: fixed as proposed, card 11 names running it from the task worktree whose plan holds the card.

## criteria-coverage

- criteria-coverage, BLOCKING, card 17's denied-recovery scan flags `go test` lines in `loom-plan-spec.md` (merged with forced-edits 3): fixed as proposed, card 17 rewords lines 362, 502 and 528, and card 18's new description never spells a listed prefix.
- criteria-coverage, MEDIUM, no guard step after a whole fallback: fixed as proposed, card 9 runs the step on a computed scope fallen back to whole, with planned rows.
- criteria-coverage, MEDIUM, guard step skip and missing-placeholder refusal untested (merged with branch-coverage 5): fixed as proposed, rows in `TestPlanTierRuns`.
- criteria-coverage, MEDIUM, Publish's publish call prunes the base entry: fixed as proposed, card 14 gives both Publish calls the base command as `BaseCommand`, with a row.
- criteria-coverage, MEDIUM, Merge-In skip untested (merged with branch-coverage 11): fixed differently, the finding asked for a landing-side row, which would test verifytree's skip again through fakes; `TestConfig_TierUse` (card 2) instead pins equal lists rendering equal calls, and verifytree's existing skip row covers the rest.
- criteria-coverage, MEDIUM, busy-hub `tree_scope_verify` and card gate call untested (merged with branch-coverage 8): fixed differently, the finding asked for held-hub rows at each site; card 8's `--wait` row now lasts past the keep-cache-warm cap, card 11's stub-`lyx` row shows the runner imposes no timeout on a gate call, and card 14 refuses a `tree_scope_verify` without `--wait`, so each half is pinned once.
- criteria-coverage, MEDIUM, `{parallel}` in a tier's GOFLAGS untested (merged with branch-coverage 6): fixed as proposed, card 8 scenario row.
- criteria-coverage, MEDIUM, planglyph's GOFLAGS untested (merged with test-economy 2): fixed as proposed, card 19 creates `goListLoader.loadEnv` with a row in `TestGlyphChain_CallerCoverageTyped`.
- criteria-coverage, MEDIUM, testtiming modes untested (merged with branch-coverage 14): fixed differently, a row per mode would run real `go test`; card 19 routes every mode through `withGateSlot`, which `TestTakeGateSlot` drives with a fake mode.
- criteria-coverage, MEDIUM, renamed failing test untested: fixed as proposed, card 10 round-scenario row.
- criteria-coverage, MEDIUM, dangling `covers` and `guard_tier` unrefused: fixed as proposed, card 8 refuses both at use, with rows.
- criteria-coverage, LOW, D2 refusals without rows: fixed as proposed, rows in cards 8 and 9.
- criteria-coverage, LOW, a tier's `dir` untested: fixed as proposed, `TestPlanTierRuns` row.
- criteria-coverage, LOW, holder `tier` and the count clamp untested (merged with branch-coverage 2): fixed as proposed, card 3 rows.
- criteria-coverage, LOW, `raw-go` steer and launcher row untested: fixed as proposed, card 17 and card 18 rows.
- criteria-coverage, LOW, `tier_parallel` in the hub file is a second source: fixed as proposed, card 2 states the entry wins from whichever layer sets it and the template comment says it is the machine override.
- criteria-coverage, LOW, slug source and non-git log location unstated: fixed as proposed, card 8 names the origin record's slug and refuses a non-git directory.

## branch-coverage

- branch-coverage, MEDIUM, `TierList` unknown gate and empty `publish` `GateCall` untested: fixed as proposed, `TestConfig_TierUse` rows.
- branch-coverage, MEDIUM, outside-hub log location untested: fixed as proposed, card 8 standalone-worktree row.
- branch-coverage, MEDIUM, `DeriveTargets` fallbacks lose coverage: fixed as proposed, card 5 carries every `TestCommandFor` row and adds nested-module and guard-scan rows, and card 10 recasts every `Derive` row.
- branch-coverage, MEDIUM, `--card` under `none` and the guard scan refusal untested: fixed as proposed, card 9 rows.
- branch-coverage, MEDIUM, `round` seam error untested: fixed as proposed, `TestVerifyGate_Scenario` row in card 10.
- branch-coverage, MEDIUM, a failure record with no tier untested: fixed differently, see the guard-reach tier finding; card 14's row writes the gate's highest tier.
- branch-coverage, MEDIUM, card 17's load-error refusal unreached: fixed as proposed, `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` rows.
- branch-coverage, LOW, config defaults and string-scalar `tier_parallel` untested: fixed as proposed, `TestLoadConfig` rows.
- branch-coverage, LOW, `AcquireSlots(…, 1)` untested: fixed as proposed, card 3 row.
- branch-coverage, LOW, `Flush` untested: fixed as proposed, card 7 row.
- branch-coverage, LOW, wiring refusal and standalone `none` untested: fixed as proposed, card 16 rows.
- branch-coverage, LOW, launcher row and validate-plan load failure untested: fixed as proposed, card 18 rows in `TestNewPlanGate` and `TestValidatePlanCmd`.
- branch-coverage, LOW, begin-batch refusal before mutation untested: fixed as proposed, `TestBeginBatch_PromptFile` row in card 15.

## row-reachability

- row-reachability, BLOCKING, loomshed's round-gate test fails at batch gates between card 8 and the old card 13: fixed differently, the finding proposed rewriting the test early or keeping `--tags`, which D2 retires; the round-gate card moved to position 10, right after the tier verb and its scopes, so no card between them targets `internal/loomshed`, and the overview records why.
- row-reachability, MEDIUM, card 13's integration rows run whatever `lyx` is on `PATH`: fixed as proposed, card 10 moves `putLyxFirstOnPath` into `lyxbin.PutFirstOnPath` and card 13 uses it.
- row-reachability, MEDIUM, card 8's flag refusals hit the config refusal first: fixed as proposed, card 8 checks flag shapes before hub resolution.
- row-reachability, MEDIUM, the two-invocation timeout row cannot happen in card 8: fixed as proposed, the row moves to card 9's per-module scope.
- row-reachability, MEDIUM, the gate-call `TestRunner` row needs a stub `lyx`: fixed as proposed, card 11 names the stub and the non-parallel `t.Setenv`.
- row-reachability, MEDIUM, cardverify tests spawn untagged: fixed as proposed, card 11 tags them `integration` and creates `testmain_test.go`, and `TestGateCardVerify` sits in gatecli's tagged file.
- row-reachability, LOW, no logger capture for the drop warning: fixed as proposed, card 2 names a non-parallel capture subtest.
- row-reachability, LOW, `TestTakeGateSlot` lacks hermetic git and a tag: fixed as proposed, card 19 edits `cmd/testtiming`'s `TestMain` and tags the test.

## forced-edits

- forced-edits, BLOCKING, removing `landingshed.Deps.GateSlots` breaks `landingDeps`: fixed as proposed, card 12 lists `loomcli#landingDeps` and envkit's `nilLegal`.
- forced-edits, BLOCKING, card 14's `publish_verify` retirement breaks `TestFinalizeVerify_Lands`, `TestLoadConfig_OptionalStringKeys` and `TestPublishVerify_HardErrorsAreReturned`: fixed as proposed, card 14 lists them.
- forced-edits, LOW, stale comments (`QuoteArg`, `prebuildLyx`, `settings.go`): fixed as proposed, cards 8, 19 and 17.

## test-economy

- test-economy, MEDIUM, `TestPlanContextFindings` duplicates `TestNewPlanGate`'s context-check table: fixed as proposed, card 18 adds rows there and drops the new test.
- test-economy, LOW, the `HolderEnv` flag rule is pinned in four places: fixed as proposed, the rule lives in `TestLeaseEnv` and the machine mapping in `TestTemplateConfigAndLimits`, and each holder keeps one pass-through row.
- test-economy, LOW, `TestTargetsFor` may drop `TestCommandFor` rows: fixed as proposed, card 5 carries every row.
- test-economy, LOW, the `TestMain`-only real-go row repeats card 7's: fixed as proposed, folded into the existing nested-module real-go row.
- test-economy, LOW, summary-line rows sit in `TestWatchdogLine`: fixed as proposed, moved to `TestWriteRunRecord`.
- test-economy, LOW, `TestGateTest_Scopes` beside `TestGateTest_Scenario`: fixed differently, the finding allowed keeping it if a fixture forbids sharing; card 9 now names that fixture, a task pair whose origin record names a parent, which the scenario's prime-only hub lacks.
