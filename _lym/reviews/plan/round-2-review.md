---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/06-tapestry-config-module.md:82
    summary: Card 6 routes the plan perch hard cap and the LoadTapestryConfig budget refusal through the tapestry spec alone, so a run-wide key (the default fallback) gets a way forward naming the wrong module
    fix: |
      In card 6, make the way forward follow the module that owns each key.
      1. perchHardCap (internal/loomcli/circling.go): pass reviewbudget.Route the spec by key, loomengine.TapestryConfigSpec() only when caps.HardKey equals reviewbudget.HardKey("plan"), else loomengine.ConfigSpec(). With the template default plan_hard_max_bounces 0, caps.HardKey is review_hard_max_bounces, a loom.yaml key, so Route over the tapestry spec would answer "lym config tapestry --layer default --set review_hard_max_bounces=<n>", a key `lym config tapestry` does not take.
      2. LoadTapestryConfig: CheckResolved takes one spec, and its hard-below-soft refusal routes both caps.HardKey and caps.SoftKey, either of which may be a run-wide loom key (a null plan_max_bounces resolves to review_max_bounces, which is the D12 acceptance case). Add to card 6 Edit the targets `internal/reviewbudget#CheckResolved`, `internal/reviewbudget#Values` and `internal/reviewbudget#TestCheckResolved`, and to Uses `internal/reviewbudget/doc.go` (doc-go-unlisted requires it). Intent: `reviewbudget.Values` gains `RunWideSpec configengine.Spec` (zero means the spec argument); CheckResolved resolves review_max_bounces, review_hard_max_bounces and review_amend_max_bounces against RunWideSpec, every perch key against spec, and each refusal names the module that owns the key it reports. LoadTapestryConfig passes tapestry's spec with RunWideSpec set to loom's ConfigSpec().
      3. Tests: a TestCheckResolved row where the plan perch inherits a run-wide soft cap above a tapestry hard cap, asserting the soft-cap route names `lym config loom` and the hard-cap route `lym config tapestry`. A TestPerchHardCap row for Plan-Bouncer with no plan_hard_max_bounces and a local loom.yaml review_hard_max_bounces, asserting the cap is that value and the raise names `lym config loom --layer local --set review_hard_max_bounces=<n>` and never `lym config tapestry`.
      Checked with `lym loom validate-plan --overlay`: the added targets pass every check.
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lym/plan/02-drop-beam-alias.md:16
    summary: lym shed refusal of a beam seed names no re-seed way forward, which the Added after Discussion entry on beam seeds requires of lym loom and lym shed alike; card 2 also still says lym loom refuses it as a non-loom recipe
    fix: |
      In card 2, give the lym shed refusal of a seed naming no known recipe the re-seed way forward the Added after Discussion entry "A beam seed names no recipe this lym knows" requires. Today shedcli's lookup answers `shedcli: unknown recipe %q; available recipes: [...]`; step closes it with the generic "fix the cause this error names, then run the step again", and status prints no way forward at all.
      Add to card 2 Edit `internal/shedcli#armFromSeed` and `internal/shedcli#TestArmFromSeed_RefusalEnvelopeKinds`, remove the latter from Uses, and add `internal/shedcli/doc.go` to Uses (doc-go-unlisted requires it). Intent: when lookup of the seed's recipe fails, armFromSeed appends `; way forward: remove the run's seed.json and seed the run again with "lym shed seed <run-id> --recipe <recipe>"`, the same re-seed route card 19 gives lym loom. The "unknown recipe on step", "unknown recipe on status" and the three loop rows of TestArmFromSeed_RefusalEnvelopeKinds assert that text in place of genericWayForward and still count one way-forward clause.
      Replace the last sentence of the Intent ("lym loom's refusal of a beam seed as a non-loom recipe arrives with card 19") with "lym loom's refusal of a beam seed as naming no recipe this lym knows, with the same re-seed way forward, arrives with card 19".
      Checked with `lym loom validate-plan --overlay`: the target changes pass every check.
  - id: F3
    severity: BLOCKING
    class: design
    location: _lym/plan/05-shed-keys-in-shed-only.md:3
    summary: The affected-set of the key removals and the tapestry move is built by hand and misses tests and docs that the cards break, card 5 leaves its own package tests uncompilable
    fix: |
      Derive each moving card's affected set from a repository search for every moved or removed key, and list every hit's enclosing test, fixture or doc. Apply these now.
      Card 5, add to Edit: `internal/loomengine#TestLoadConfig_Loads` (its explicit arbiter keys, arbiter_max_spawns 0, driver_kind and tackler rows assign the Config fields c.Arbiter, c.ArbiterTimeoutMin, c.ArbiterMaxSpawns, c.DriverKind and c.Tackler* that the card deletes, so the package's tests stop compiling at card 5, and its local driver_kind row now expects a refusal) and `internal/darnengine#TestMigrateConfig` (its withVerify fixture inserts tree_scope_verify before the template line driver_kind:, which the card removes, so the fixture silently inserts nothing; anchor it on another key). Remove `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag` from card 5 Uses (uses-later-target once card 6 edits it).
      Card 6, move `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag` from Uses to Edit (the card gives the moved fields the tag yaml:"-", and the test fails any field whose tag is not a template key; the seam-test-unlisted check also demands the test be covered), and add to Edit: `internal/loomengine#TestConfigTemplate_ImplementerDefaultsToWebsterAndAcceptsALocalValue` (asserts implementer in loom's template and ConfigSpec), `internal/loomengine#twillLenses` (its errors name "loom config key" for plan_lenses, which is now a tapestry key), `internal/loomcli#TestWire_ReviewKeysReachTheEnv` (seeds plan_reviewer_model, plan_judge, plan_max_bounces, plan_hard_max_bounces, plan_reviewer_fan and plan_fixer_fan into loom.yaml, expects the Plan-Review segment's ConfigFile loom.yaml and loom module keys, and reads env.Setting("loom","plan_reviewer_fan")), `internal/loomcli#seedLoomConfigWithKeys` (it fatals on a key absent from loom's template and only writes loom.yaml; give it a tapestry form), `internal/configreg#TestLayerRunSettings` (its plan_max_bounces rows expect the document under loom, KeyMoves now translates it to tapestry), `internal/configreg#TestLookup` (loom's open-map list names plan_reviewer_model, plan_fixer_model, plan_reviewer_fan and plan_fixer_fan), `internal/configcli#TestPrintResolved` (expects plan_timeout_min in the loom listing), `internal/configcli#TestConfigOverRealHub` (plants plan_reviewer_fan and plan_fan in the board loom.yaml and expects plan_reviewer_fan there after reconcile), `internal/hubreconcile_test#loomFormerUnits` (formerUnits fails the test on plan_lenses, plan_reviewer_model and the plan fan keys once loom's template drops them; move them to a tapestry case of preMigrationBase) and `contracts/specs/llm-model-spec.md` (its loom.yaml paragraph lists plan_judge and plan_seed).
      Checked with `lym loom validate-plan --overlay` together with F1: the combined overlay reports no blocking finding.
  - id: F4
    severity: LOW
    class: design
    location: _lym/plan/10-drift-verify-before-review.md:12
    summary: Card 10 creates TestRecipe_DriftVerifyPrecedesCodeReview for edges the existing TestRouting_ProgressAt tail subtest already pins and the card edits anyway
    fix: |
      In card 10, delete the Create section and the paragraph that describes `TestRecipe_DriftVerifyPrecedesCodeReview`. State instead that the tail subtest of `internal/loomrecipe#TestRouting_ProgressAt`, already an Edit target, asserts the new edges row by row: Parent-Align to Drift-Verify, Drift-Verify to Code-Bouncer, Code-Bouncer on_done to Describe, and Finalize's reroute to Parent-Align, from which Drift-Verify follows by on_done. `TestRowNames_FollowEachRecipesRowOrder` and `wantProducerTable` already pin the file order.
      Checked with `lym loom validate-plan --overlay`: removing the Create section passes every check.
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/plan/17-one-describe.md:11
    summary: Edit target runRecordBullets lacks the re-sign arrow although the Intent gives it an error result
    fix: |
      In card 17 Edit, replace `internal/landingshed#runRecordBullets` with `internal/landingshed#runRecordBullets` -> `func runRecordBullets(in DescribeInputs) (string, error)`, written as card 13 writes its arrow targets. The Intent has runRecordBullets skip absent paths and DescribeTable return a stat error other than not-exist, and today it returns a bare string.
      Checked with `lym loom validate-plan --overlay`: the arrow target passes every check.
  - id: F6
    severity: LOW
    class: design
    location: _lym/plan/18-recipe-keyed-wiring.md:38
    summary: TestRecipeKeyedWiring repeats the per-recipe landingDeps EscalateJudgedFirst assertions that TestWire_BeamRecipe already makes
    fix: |
      In card 18, drop the sentence that has TestRecipeKeyedWiring call landingDeps per recipe and assert EscalateJudgedFirst for tapestry, darn and pawl. The subtest "the landing deps of a pawl run halt a spent fixer, and only a tapestry or darn run judges an escalate first" of `internal/loomcli#TestWire_BeamRecipe` (wiring_beam_test.go) already asserts EscalateJudgedFirst for those three recipes and pawl's nil TraceFailure and empty PlanDir. Keep only the TraceFailure-set assertion for tapestry in TestRecipeKeyedWiring, the one landingDeps fact no test pins. Do not move the assertions into TestWire_BeamRecipe: cards 9, 11, 13 and 15 list it under Uses, so a later Edit would be refused.
  - id: F7
    severity: MEDIUM
    class: decision
    location: _lym/plan/16-decision-add-by-rows.md:36
    summary: Card 16 names the goto to Record-Write for a record not committed on a later row, where D4 and its acceptance criterion say only lym loom resume, and the decision record carries no Added after Discussion entry for the departure
    fix: |
      Not a card edit. The parent records the departure with `lym loom decision add`: an entry that amends D4's Not-committed bullet and the D4 acceptance clause to the row-dependent text card 16 already carries (current row Record-Write names `lym loom resume`; any later row names `lym loom goto --to Record-Write`, then `lym loom resume`, then re-running the verb), with the reason that a darn run in flight that halted at Darn-Write on its old discussion gate also holds a written, uncommitted record. Card 16's Intent then cites the entry. Until the parent records it, the fixer defers this finding; it must not revert card 16 to the bare `lym loom resume`, which repeats forever on that run.
---

### [BLOCKING] Card 6 misroutes run-wide keys in the plan perch's hard-cap way forward and budget refusal

**Location:** `_lym/plan/06-tapestry-config-module.md:76` and `:82`, against `internal/loomcli/circling.go:195` and `internal/reviewbudget/reviewbudget.go:83,145`.

**Issue:** Focus 3 asks whether a tuned `tapestry.plan_hard_max_bounces` still reaches `perchHardCap`; it does, because card 6 loads `LoadTapestryConfig` and applies `Config.WithTapestry` before `Caps`. The defect is in the way forward. `reviewbudget.Resolve` returns `HardKey` `review_hard_max_bounces` whenever the perch's own hard cap is 0, which is the template default, and `Route` resolves the layer from the one spec it is given. Card 6 routes the plan perch's way forward "through `TapestryConfigSpec()`", so the default configuration yields `lym config tapestry --layer default --set review_hard_max_bounces=<n>`, naming a key the tapestry module does not hold. The same single-spec shape sits in `CheckResolved`: D12's acceptance case, a hard cap below the budget inherited from `loom.review_max_bounces`, routes `caps.SoftKey` `review_max_bounces` through the tapestry spec and prefixes the message with `tapestry config key`. PATTERN-refusal-way-forward requires a loom refusal to name the way forward; a named way forward that the config CLI refuses is not one. The card's `TestPerchHardCap` row seeds only the tapestry-owned key, so no test reaches the run-wide case.

### [BLOCKING] The lym shed refusal of a beam seed carries no re-seed way forward

**Location:** `_lym/plan/02-drop-beam-alias.md:16` and `:18`, against the `Added after Discussion` entry "A beam seed names no recipe this lym knows" and `internal/shedcli/cli.go:264`.

**Issue:** Focus 4 asks whether card 19's seed wording matches the record. For `lym loom` it does: card 19 refuses a `beam` seed as naming no recipe this lym knows and names removing `seed.json` and re-seeding, which is what the entry says. The entry says `lym loom` and `lym shed` alike refuse it as naming no embedded recipe, and "the refusal names the re-seed way forward". Card 2 leaves `lym shed` on shedcli's existing `unknown recipe %q; available recipes` error. `armFromSeed` passes that error through unchanged, `step` appends only the generic "fix the cause this error names", and `status` prints no way forward (the "unknown recipe on status" row of `TestArmFromSeed_RefusalEnvelopeKinds` asserts no suffix). The record carries no entry accepting that departure, and the entry's own rationale is that a re-seed is the only way forward that works. Card 2's last Intent sentence also still describes the superseded wording ("refusal ... as a non-loom recipe"), which the entry replaces.

### [BLOCKING] Key removals and the tapestry move leave tests uncompilable or failing, and the card lists miss them

**Location:** `_lym/plan/05-shed-keys-in-shed-only.md:3` (Edit list) and `_lym/plan/06-tapestry-config-module.md:14` (Edit list).

**Issue:** Focus 5 asks whether every reader, fixture, doc and background file naming the moved keys is a target. It is not, and one gap stops card 5 itself. `internal/loomengine/config_test.go` `TestLoadConfig_Loads` assigns `c.Arbiter`, `c.ArbiterTimeoutMin`, `c.ArbiterMaxSpawns`, `c.DriverKind` and `c.Tackler*`, fields card 5 deletes, so loomengine's tests stop compiling at card 5 and the card gate is red; card 6 edits that test one card too late. `internal/darnengine/config_test.go` `TestMigrateConfig` builds its fixture with `strings.Replace(..., "driver_kind:", ...)`, an anchor card 5 removes. Card 6 lists `TestConfigTemplate_ContainsEveryConfigYAMLTag` under Uses and says the plan fields keep the tag `yaml:"-"`, but that test errors on every `Config` field whose tag is not a template key, and the `seam-test-unlisted` check confirms the test must be covered; card 5 also Uses it, which becomes `uses-later-target` once card 6 edits it. The remaining misses are the tests and docs named in the fix: the other loomengine, loomcli, configreg, configcli and hubreconcile tests that name `plan_*` or `implementer` in loom's module, `twillLenses`' error naming the loom module for `plan_lenses`, and the loom paragraph of `contracts/specs/llm-model-spec.md`. The pattern is a hand-built list that follows the symbols the author remembered, so it is filed once as a method finding. Already covered and not re-raised: Tapestry-Preflight and `CheckSeed` readers (every hit in the repository is a card 9 target, bar a header comment), the recipe `module.key` references, and `docs/overview.md`.

### [LOW] Card 10 adds a test that the edited TestRouting_ProgressAt already makes

**Location:** `_lym/plan/10-drift-verify-before-review.md:12`.

**Issue:** `internal/loomrecipe#TestRouting_ProgressAt`'s tail subtest (progress_test.go) asserts `OnDone`, `OnStuck` and `OnReroute` row by row for Parent-Align, Code-Bouncer, Drift-Verify, Describe, PR-Review and Finalize, including Finalize's reroute to Parent-Align. Card 10 already edits that test "to follow the new row order", which makes it assert exactly the edges the new `TestRecipe_DriftVerifyPrecedesCodeReview` walks. The extra Finalize-reroute claim follows from the same table plus Parent-Align to Drift-Verify. A second test for the same behavior is a defect.
The first round settled this card's test as pinning edges "which no existing test pins", checking only `TestNew_ShapeMatchesRecipe`; that conclusion does not hold, because `TestRouting_ProgressAt`'s tail subtest asserts those edges exactly and the card edits it.

### [LOW] runRecordBullets changes its result without the re-sign arrow

**Location:** `_lym/plan/17-one-describe.md:11`.

**Issue:** `runRecordBullets(in DescribeInputs) string` renders paths without touching disk. Card 17 makes it skip paths that do not exist and has `DescribeTable` return "a stat error other than not-exist", which a bare string cannot carry, so the result type changes. The Edit target is listed without the arrow that card 13 and card 16 use for their changed signatures.

### [LOW] Card 18 repeats an existing per-recipe landingDeps assertion

**Location:** `_lym/plan/18-recipe-keyed-wiring.md:38`.

**Issue:** `TestRecipeKeyedWiring` "also calls `landingDeps` per recipe: `EscalateJudgedFirst` holds for tapestry and darn and not for pawl". The `internal/loomcli#TestWire_BeamRecipe` subtest at `wiring_beam_test.go` builds `landingDeps` for those three recipes and asserts `EscalateJudgedFirst` per recipe, plus pawl's nil `TraceFailure` and empty `PlanDir`. The first assertion adds nothing; the tapestry `TraceFailure` assertion is the one new fact.
The first round settled the card as covering "branches no existing loomcli test drives"; that holds for `verifySource`, `fixerPlanDir` and the tapestry `TraceFailure`, and not for `EscalateJudgedFirst`.

### [MEDIUM] Card 16's not-committed way forward departs from D4 with no recorded decision

**Location:** `_lym/plan/16-decision-add-by-rows.md:36`, against `_lym/discussion/decision-record.md` D4 ("Not committed: only a crash inside Record-Write leaves this ... the way forward is `lym loom resume`") and the D4 acceptance clause ("`decision add` refuses a darn record not committed at HEAD naming `lym loom resume`").

**Issue:** This is the first round's deferred item F8, re-evaluated. The fixer rewrote card 16 so that on a row after Record-Write the refusal names `lym loom goto --to Record-Write`, then `lym loom resume`, and deferred the decision-record entry to the parent. The record still carries only the two `Added after Discussion` entries on the build segment and the beam seed, so the card and the answer key still disagree on both the premise (a crash inside Record-Write is the only cause) and the named step. The card's text is the one that works: a darn run in flight parked on Darn-Write with a record written by the old writer is uncommitted at HEAD, and a bare `lym loom resume` re-enters Darn-Write, which after cards 13 and 15 commits only the landing directory. Focus 2's question is answered by this: the named step commits the record, and the remaining gap is the missing decision, which the parent alone can record.

## Rejected

- Focus 1, card order around the recipe-keyed builder. Card 8's `New(recipe, …)` stamps `DarnVerifyAttempts` on a recipe whose build row engine is `DarnWrite`, which holds for darn at cards 8, 9 and 10; card 11 switches the stamp to `verify_source: tree_scope` in the same card that gives `darn-recipe.yaml` the keys. `New` refuses no recipe for its contract before card 11, and card 7's accessors are only called from card 9 on. No earlier card's tests or builds for darn fail in between. Not a defect.
- Focus 2, the not-committed way forward. A darn run parked on Darn-Write with a written, uncommitted record reaches `checkDarnRecord` with `committed()` found false; the named step, `lym loom goto --to Record-Write` then `lym loom resume`, runs Record-Write, whose `Commit` is `commitDiscussion` over the whole discussion directory and so commits the pair; Darn-Write's landing-directory commit and the post-session guard never needed to. The refusal has named inputs and rows in `TestDecisionVerb_Refusals` (current row Record-Write naming `resume`, current row Darn-Write naming the goto). The recovery step is not a defect; the missing decision-record entry is F7. A run past Darn-Write would redo rows by the goto where `lym fabric commit` (already named by the differs-from-committed refusal) would also commit it; that is a choice for the parent's entry in F7.
- Focus 3's first half, whether a tuned `tapestry.plan_hard_max_bounces` reaches the read. It does, through `LoadTapestryConfig` and `Config.WithTapestry`; the routing half is F1.
- Retired shed keys: card 5 keeps them out of `retiredLoomKeys` and `MigrateConfig`, declares them with `CategoryGuard`, and refuses them in run settings and local files with one Spec text, as D13 states. No defect found.
- Contract engine sets: `shedbuild` already imports `shedrecipe`, so card 3's sets beside the registry introduce no cycle.
