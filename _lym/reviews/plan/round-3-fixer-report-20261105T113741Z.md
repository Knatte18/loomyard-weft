# Round 3 fixer-report — plan

## Changes

- **F1** (BLOCKING), fixed as proposed: `_lym/plan/07-report-config.md` now creates the module in `internal/reportconfig`.
  Its **Create:** list holds `doc.go`, `testmain_test.go`, `template.yaml`, and `#Config`, `#ConfigTemplate`, `#ConfigSpec`, `#LoadConfig` and `#TestLoadConfig` with their signatures unchanged.
  The `internal/reportengine#allowedImports` edit and its Intent sentence are gone.
  The Intent gives the import-cycle reason, the `configreg.Modules` row over `reportconfig`, and the line "`testmain_test.go` is untagged, the package's `TestMain`."
- **F2** (BLOCKING), fixed as proposed: `_lym/plan/08-report-cost.md` lists `plan:internal/reportconfig#LoadConfig` under **Uses:**, and its `costCommand` paragraph reads `cost_min_runs` with `reportconfig.LoadConfig`.
- **F3** (MEDIUM), fixed as proposed and folded into the card 7 rewrite: `internal/configreg#TestNames` and its Intent clause are dropped.
  `internal/configreg#TestModules_SeedOnlyAndHubWideFlags` and `cmd/lym#configStrictnessDegradingSet` are added under **Edit:**, each with one Intent line.
- **F4** (MEDIUM), fixed as proposed.
  In `_lym/plan/23-raddle-recipe.md`, the `loomcli`, `shedrun`, `loomrecipe`, `shedcli` and `runprofile` doc.go files move to **Edit:**, and only `internal/shedrecipe/doc.go` stays under **Extends:**.
  The Intent's `loomcli/doc.go` sentence now rewords the motif-only Sketch sentence.
  A new sentence names the recipe lists each of the other four doc files gains `raddle` in, with `light` in the `runprofile` defaults sentence.
  In `_lym/plan/25-pawl-raddle-handoff.md`, `internal/pawlengine/doc.go` moves to **Edit:** and the empty **Extends:** section is removed.
  An Intent sentence adds the hand-off marker to its `.lym/beam/` file list and the hand-off step to its Beam-Up description.

`lym loom validate-plan --overlay` over the four changed cards reports only `create-already-exists` blocking findings on the landed cards 7 to 12, the baseline pattern the review names; nothing else blocks.
The round's gate reports the same findings; see Deferred.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- **Gate: `create-already-exists` on cards 7 to 12** (blocking), deferred: no plan change resolves it soundly.
  Each finding names a Create target that cards 7 to 12 landed in commits `5ec85b9c` to `d7f0465e`, so the plan is right and the targets exist because the work is done.
  `lym loom validate-plan --help` says the gate skips the tree-dependent checks only for "cards of batches webster's run record holds done".
  Batch 7 failed, so the run record does not hold these cards done.
  Turning those cards' Create targets into Edit targets would silence the gate but rewrite six landed cards to describe work they did not do.
  It would also break Webster's done-checks for the batch, which test that each Create target is present.
  The repair sits in Webster's run record: re-drive the failed batch through `record-batch` or `recover-batch`, as `internal/websterengine/beginbatch.go`'s `DispatchScope` describes.
  That is outside this fixer's write surface, which is `_lym/plan` and the round's report files.
  The two `create-new-unit` findings are informational and need no change.
