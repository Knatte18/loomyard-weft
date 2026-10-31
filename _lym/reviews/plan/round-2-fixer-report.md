# Round 2 fixer-report — plan

## Changes

- F1 (BLOCKING), fixed as proposed, card 6.
  `perchHardCap` routes the hard-cap way forward through the module that owns `caps.HardKey`: `TapestryConfigSpec()` for `plan_hard_max_bounces`, `ConfigSpec()` otherwise.
  `reviewbudget.Values` gains `RunWideSpec`, and `CheckResolved` routes and names the run-wide keys through it; `LoadTapestryConfig` sets it to loom's `ConfigSpec()`.
  Added `internal/reviewbudget#Values`, `#CheckResolved` and `#TestCheckResolved` to Edit and `internal/reviewbudget/doc.go` to Uses.
  New `TestCheckResolved` row and second `TestPerchHardCap` Plan-Bouncer row, as the fix specified.
- F2 (BLOCKING), fixed as proposed, card 2.
  `armFromSeed` and `TestArmFromSeed_RefusalEnvelopeKinds` are Edit targets, and `internal/shedcli/doc.go` is under Uses.
  On a failed recipe lookup, `armFromSeed` appends the re-seed way forward naming `lym shed seed <run-id> --recipe <recipe>`.
  Every test row that seeds an unknown recipe asserts it: both "unknown recipe" rows and the four `loop:` rows seeding `bogus-recipe`, not three as the fix said.
  The last Intent sentence now describes card 19's no-known-recipe refusal.
- F3 (BLOCKING), fixed as proposed, cards 5 and 6.
  - Card 5 Edit gains `internal/loomengine#TestLoadConfig_Loads` and `internal/darnengine#TestMigrateConfig`; the latter re-anchors its fixture on `verify_attempts:`.
  - `TestConfigTemplate_ContainsEveryConfigYAMLTag` leaves card 5's Uses.
  - Card 6 moves `TestConfigTemplate_ContainsEveryConfigYAMLTag` to Edit; it now skips `-` tags and checks `TapestryConfig`'s tags against the tapestry template. Card 6 Edit also gains every other target the fix named, with an Intent line each.
  - Fixed differently in two places:
    - `twillLenses` gains a `module` parameter (arrow target), and its two callers, `planLensDirective` and `discussionLensDirective`, join Edit. The error can name the owning module only if a caller passes it in.
    - `seedLoomConfigWithKeys` writes each key to whichever module's template holds it, rather than gaining a separate tapestry form. Its existing callers then stay unchanged.
  - `preMigrationBase` joins Edit for its new `tapestry` case.
- F4 (LOW), fixed as proposed, card 10.
  Create section and `TestRecipe_DriftVerifyPrecedesCodeReview` removed. The Intent says `TestRouting_ProgressAt`'s tail subtest asserts the new edges.
- F5 (LOW), fixed as proposed, card 17.
  `runRecordBullets` carries the arrow `func runRecordBullets(in DescribeInputs) (string, error)`.
- F6 (LOW), fixed as proposed, card 18.
  `TestRecipeKeyedWiring` keeps only the tapestry `TraceFailure` assertion and cites `TestWire_BeamRecipe` for the rest.

`lym loom validate-plan` passes after all changes, with only the informational `create-new-unit` finding for `tapestry.yaml`.

## Self-check

- fix-consistency, LOW, card 2 ImpactSummary stale: fixed as proposed (names the new way forward on every unknown-recipe refusal).
- fix-consistency, LOW, card 2 calls its route card 19's "same" text while it names `lym shed seed` alone: fixed with my own change, rewording the claim as "the `lym shed` form of" card 19's route rather than copying card 19's text. Card 19's `lym loom start` alternative does not fit a `lym shed` refusal of an arbitrary unknown recipe.
- fix-consistency, LOW, card 6 ImpactSummary stale: fixed as proposed.
- fix-claims, MEDIUM, card 6 `TestConfigOverRealHub` planted `plan_fan` in `tapestry.yaml`, which has no rename for it: fixed as proposed. The test keeps planting in `loom.yaml` and expects `plan_reviewer_fan` in `tapestry.yaml` after the rename and move.
- fix-claims, LOW, card 2 loop path unverified: disputed, premise false. `ReportLoopArmError` builds its message with `bootstrapMessage(err)` (`internal/shedverbs/loopenvelope.go`). The generic way forward is appended only when the text names none (`internal/shedverbs/step.go:82-85`).
- fix-proposal: no findings.
- fix-affected-set, MEDIUM, card 5 misses `internal/loomcli#seedGoDriverRun`, which writes `tackler_max_spawns` and `arbiter_max_spawns` into the local `loom.yaml` (`smoke_helpers_test.go:186-190`): fixed as proposed. It is added to card 5's Edit list and now writes the local `shed.yaml`.
- fix-affected-set, LOW, `TestResolveReviewAndJudge_MalformedSpec` and `TestPlanTable` may assert loom's module: disputed, premise false.
  - The `plan_judge`/`plan_seed` rows of `TestResolveReviewAndJudge_MalformedSpec` (`internal/loomengine/review_test.go:300-345`) name no `loom` module or file.
  - `TestPlanTable`'s lens-fan rows (`internal/loomengine/plan_test.go:444-448`) assert `plan_lenses`, the fan, `lenses.yaml` and `set the key to ""`, never `loom config key`.

## Disputed

- Self-check fix-claims LOW (card 2 loop path): premise false; evidence above.
- Self-check fix-affected-set LOW (`TestResolveReviewAndJudge_MalformedSpec`, `TestPlanTable`): premise false; evidence above.

## Deferred

None.
