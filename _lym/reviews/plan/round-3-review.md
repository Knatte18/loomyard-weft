---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/09-drop-tapestry-preflight.md
    summary: The affected-set search follows config key literals but misses tests that read a changed value back through a loader or a derived number, so cards 6, 9, 10 and 11 each leave a failing test outside their targets
    fix: |-
      Extend the method the plan writer uses to build each card's affected set, and apply the instances below now.
      Besides a repository search for every moved or removed key, search for each reader of a Go field the card stops filling (here `Config.PlanLenses` and the other plan fields, read back after `LoadConfig`) and for each test or fixture that pins a value derived from the changed rows (here the numbers `loomrecipe.EntryNumbers` gives each row), and list every hit's enclosing test as an Edit target of the card that breaks it.
      Card 06 gains `internal/loomengine#TestMigrateConfig` (card 05 already edits it for the shed-override rows) and one Intent sentence saying the "migrated layers load with the advisors each layer's migration left" subtest drops its `PlanLenses == "twill-plan"` conditions and error-message operands, because `LoadConfig` leaves the plan fields at their zero value and only warns on the migrated `plan_lenses` key; the `plan_producer` to `plan_lenses` rewrite stays pinned by the table rows above that subtest.
      Card 09 (Tapestry-Preflight leaves, so every tapestry number after Preflight drops by one and Plan-Write becomes 5) gains `internal/loomrecipe#TestEntryNumbers`, `internal/battencli#TestOverviewRunCells`, `internal/battencli#TestDobbyRunCells_ReadsTheRunsFromDisk`, `internal/battencli#TestDobbyRunCells_ResolvesTheChildPairAgainInEachRunShedSpell` and `internal/battencli#TestDobbyLines`, with an Intent sentence saying the tapestry map of `TestEntryNumbers` renumbers and the `6.6 Plan-Write` expectations become `6.5 Plan-Write`.
      Card 10 (Drift-Verify moves ahead of Code-Bouncer) gains `internal/loomrecipe#TestEntryNumbers`.
      Card 11 (Record-Write joins darn after Preflight, so Drift-Baseline becomes 3 and every later darn number shifts) gains `internal/loomrecipe#TestEntryNumbers` and `internal/loomcli#TestRenderChecklist`, with an Intent sentence naming the darn map and the checklist rows to renumber.
      The combined overlay of all these target additions ran through `lym loom validate-plan --overlay` and reported nothing beyond the baseline.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/06-tapestry-config-module.md
    summary: No named test reaches the hard-below-soft refusal when the plan hard cap is unset and resolves to the run-wide key
    fix: |-
      In card 06's Tests paragraph, extend the `TestCheckResolved` sentence with a second row.
      The row has `RunWideSpec` set, a plan perch with its own `plan_max_bounces` above the run-wide `review_hard_max_bounces` and `plan_hard_max_bounces` at 0.
      It asserts the refusal's prefix reads `loom config key "review_hard_max_bounces"`, the raise-the-hard-cap route names `lym config loom`, and the lower-the-soft-cap route for `plan_max_bounces` names `lym config tapestry`.
      The existing row covers only the mirror case, a run-wide soft cap above the plan perch's own hard cap.
      This is card-text only: no target or verify line changes, so no overlay is needed.
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/plan/02-drop-beam-alias.md
    summary: Card 2 has the unknown-recipe status row assert the re-seed suffix, but the test's kind-less branch returns before any suffix assertion
    fix: |-
      In card 02's Intent, replace "the status row gaining one" with a sentence saying the kind-less branch of `TestArmFromSeed_RefusalEnvelopeKinds` (the `wantKind == ""` early return) also asserts `wantSuffix` on the envelope's error text when the row sets one, and that the "unknown recipe on status" row then sets `wantSuffix` to the re-seed text.
      Without that harness change the status row passes without ever checking the suffix.
      Card-text only, so no overlay is needed.
---

### [BLOCKING] The affected-set method still misses tests that read a changed value indirectly

**Location:** `_lym/plan/06-tapestry-config-module.md`, `_lym/plan/09-drop-tapestry-preflight.md`, `_lym/plan/10-drift-verify-before-review.md`, `_lym/plan/11-record-write-row.md`

**Issue:** Round 2 filed the hand-built affected-set once as a method finding, and the fix searched the repository for every moved or removed key literal.
That search cannot find a test that reads the change through something other than the key text, and two such classes remain.
First, `internal/loomengine/config_test.go#TestMigrateConfig` ends with the subtest "migrated layers load with the advisors each layer's migration left", which migrates `oldFormatLoomYAML` (its `plan_producer: twill` becomes `plan_lenses: twill-plan`), loads it with `LoadConfig` and fails unless `got.PlanLenses == "twill-plan"`, in two assertions.
Card 6 keeps the `Config` plan fields tagged `yaml:"-"` and states "`LoadConfig` never reads `tapestry.yaml`", and `warnUnknownKeys` in `internal/configengine/config.go` only warns on a base-layer key the template lacks.
After card 6 the load succeeds, `PlanLenses` is empty and the subtest fails; card 5 lists `TestMigrateConfig` for its shed-override rows, and card 6 does not list it.
Second, `loomrecipe.EntryNumbers` numbers each row of the shipped recipe, and several tests pin the result.
`internal/loomrecipe/recipe_test.go#TestEntryNumbers` pins the tapestry map (`"Preflight": 1, "Discussion-Write": 3`, the gap being Tapestry-Preflight, `"Code-Bouncer": 11`, `"Drift-Verify": 12`) and the darn map (`"Drift-Baseline": 2` through `"Code-Bouncer": 6`).
`internal/battencli/overview.go#overviewEntryNumber` reads the same numbers from the real tapestry recipe, so `TestOverviewRunCells`, `TestDobbyRunCells_ReadsTheRunsFromDisk`, `TestDobbyRunCells_ResolvesTheChildPairAgainInEachRunShedSpell` (all in `overview_test.go`) and `TestDobbyLines` (`overviewdobby_test.go`) expect `6.6 Plan-Write`, Run-Shed's 6 plus Plan-Write's 6 while Tapestry-Preflight holds number 2.
`internal/loomcli/checklist_test.go#TestRenderChecklist` builds `darnRows` from `loomrecipe.Rows(shedrun.RecipeDarn)` and pins `[ ] 2. Drift-Baseline` through `[ ] 10. Friction-Reflect`.
Card 9 deletes Tapestry-Preflight, card 10 moves Drift-Verify ahead of Code-Bouncer, and card 11 inserts Record-Write after darn's Preflight; each shifts numbers these tests assert.
None of the seven tests appears in any card's targets (`grep` over `_lym/plan/[0-9]*.md` finds no mention of `TestEntryNumbers`, `overview_test` or `checklist`), so each card's gate (tier0, card scope) fails on a test outside the card's write set.
The writer tracked the row order in prose and in `wantProducerTable`, but not the numbered consumers.
The `class` is `design` because the gap is the method's, recurring from rounds 1 and 2; the instances above are what the corrected method finds now.

### [MEDIUM] The run-wide hard cap beneath a plan perch's own soft cap has no reaching test

**Location:** `_lym/plan/06-tapestry-config-module.md`, Tests paragraph

**Issue:** The focus question was whether, when the plan perch's own hard or soft cap is unset and resolves to a run-wide key, each way forward and refusal names the module owning the key it reports, and whether a named test reaches that case.
The prescription is right and general: `CheckResolved` routes `review_max_bounces`, `review_hard_max_bounces` and `review_amend_max_bounces` through `RunWideSpec`, names the owner module in each prefix, and `perchHardCap` picks `TapestryConfigSpec()` only when `caps.HardKey == reviewbudget.HardKey("plan")`.
Two named tests reach the soft-unset direction: the `TestCheckResolved` row (run-wide soft cap above the plan perch's own hard cap) and the `TestLoadTapestryConfig` row (`plan_hard_max_bounces` below the budget inherited from `loom.review_max_bounces`).
`TestPerchHardCap`'s second Plan-Bouncer row reaches the hard-unset direction, but only `perchHardCap`'s route choice with a passing cap.
No named test reaches the refusal in `CheckResolved`'s final loop when the hard key is run-wide (`review_hard_max_bounces`, `plan_hard_max_bounces: 0`) and the soft key is the plan perch's own `plan_max_bounces`.
That refusal is the one that must print the loom prefix and `lym config loom` for the hard route beside `lym config tapestry` for the soft route; the rubric counts a prescribed refusal branch with no reaching test as a finding.
`internal/loomcli/circling.go#perchHardCap` and `LoadTapestryConfig` were checked against PATTERN-refusal-way-forward and are otherwise consistent with it.

### [LOW] The status row of the unknown-recipe refusal cannot assert the re-seed suffix

**Location:** `_lym/plan/02-drop-beam-alias.md`

**Issue:** Card 2 says every row of `TestArmFromSeed_RefusalEnvelopeKinds` that seeds an unknown recipe "asserts that text as its suffix, the status row gaining one".
In `internal/shedcli/cli_test.go` the "unknown recipe on status" row sets no `wantKind`, and the harness returns after checking the envelope has two keys, before it reaches the `HasSuffix` and way-forward-count assertions.
The step and loop rows do reach the suffix, so the shed refusal's re-seed way forward is tested on those paths, and the wording matches the decision record's beam-seed entry (a refusal naming the re-seed way forward).
Only the status row, as described, would pass without asserting it.

## Rejected

- Card 16, not-committed refusal: the card's row-dependent way forward (`lym loom resume` at Record-Write, `lym loom goto --to Record-Write` then `lym loom resume` then re-run on a later row) agrees with the decision record's "A record not committed names its way forward by the current row" entry, and its `TestDecisionVerb_Refusals` rows reach both rows. Not a defect; the round 2 finding on it is resolved by the record's entry.
- Card 10: it edits the existing tail subtest of `TestRouting_ProgressAt` row by row and adds no test, so it repeats nothing that test already asserts. Not a defect (the numbering consumers are in F1).
- Card 17: the Edit list gives `runRecordBullets` the re-sign arrow with `(string, error)`; the current result is `string`. Not a defect.
- Card 18: `TestRecipeKeyedWiring` leaves `EscalateJudgedFirst` to `TestWire_BeamRecipe` and adds only branches no test covers (`verifySource`, `arbiterRecipeDescription`, a tapestry `TraceFailure`, which `landingdeps_test.go` asserts only as nil for darn). Its pawl `fixerPlanDir` assertion overlaps the `deps.PlanDir` check in `TestWire_BeamRecipe`, one assertion among otherwise uncovered ones. Not raised.
- Card 5 and card 6 key search over `internal/darnengine`, `internal/loomengine`, `internal/loomcli`, `internal/configreg`, `internal/configcli` and `internal/hubreconcile`: every other hit on a removed or moved key (`TestPrepareRunSettings`, `TestLayerRunSettings`, `TestSettingLookup`, `TestLookup`, `TestModules_*`, `TestDispatchLayers`, `TestPrintResolved`, `TestConfigOverRealHub`, `TestBuild_*`, `TestFullEnv_FillsEverySeam`, `TestWire_ReviewKeysReachTheEnv`, `seedLoomConfigWithKeys`, `tacklerFor`, `childDriverKind`, `HoldReading`, `seedGoDriverRun`, `RecipeOverride`) is an Edit or Delete target of the card that breaks it. The hubreconcile integration test's `driver_kind` hits are shed's own key, and its `loomFormerUnits` and `preMigrationBase` are targets of card 6.
- `seedLoomConfigWithInteractive` and `seedLoomConfigWithFriction` in `internal/loomcli/wiring_test.go`, and the webstercli copy, write `plan` and `plan_timeout_min` into `loom.yaml`; after card 6 those are base-layer keys the template lacks, which warn and load. Not a break.
- The `git -C _lym checkout HEAD -- discussion/decision-record.md` way forward in card 16 uses a form the closed denied list does not hold (`git checkout <head> -- <path>` is named valid), and `_lym` resolves to the weft repository here. Not a defect.
- Round 2's disputed items (the card 2 loop path, `TestResolveReviewAndJudge_MalformedSpec` and `TestPlanTable`) are not re-raised.
