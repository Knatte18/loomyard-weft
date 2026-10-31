```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change a card makes, I traced what else reads or asserts it, and checked which cards cover that.

**Per change: what reads it, and which cards cover it**

- **Card 6, recipe name `sampler` added to `shedrun.RecipeNames`.**
  - Readers: `shedrun/seed.go:89-94` (ValidateRecipe), `shedrun/seed_test.go:195`, `shedcli/table_test.go:31-41`, `cmd/lym/driverstencil_test.go:30` (`TestDriverStencil_IsRecipeBlind`).
  - Covered by card 6, except the driver-stencil test. No card lists it, but it passes unchanged as long as the driver stencils never name `sampler`, so it is held.
- **Card 6, `Sampler-Session` added to `loomshed.InterruptPolicies`.**
  - Readers: `TestInterruptPolicies_MatchAssembledRows` (covered by card 6) and `cmd/lym/driverstencil_test.go:37` (the row-name scan, held unchanged).
- **Cards 5 and 6, registry key `SamplerSession`.**
  - Readers: `shedrecipe/coverage_guard_test.go:51` `TestCoverageGuard`, which unions `loomrecipe.RecipeEngines()`.
  - Covered by card 6's `RecipeEngines` edit. Card 5 lands the key first. That is a card-order question, left to the repo-invariants lens.
- **Card 5, way-forward text of `checkLoomContract`.**
  - Readers: `shedbuild/build_test.go:176,181`. Covered by card 5. I did not confirm the enclosing test's name.
- **Card 6, new `RecipeRow.OnDone` field.**
  - Readers: `TestRowNames_FollowEachRecipesRowOrder`. Covered by card 6.
- **Card 8, new signatures of `codeBudgetZero` and `rerouteReviewed`.**
  - Readers: `landingdeps.go:82,272` and `landingdeps_test.go:531-532` (`TestRerouteReviewed_ReadsTheCodeBudgetFromLoomYaml`, line 513). Covered by card 8.
- **Card 12, new signature of `agentActivityFor`.**
  - Readers: `arm.go:431` (specFor) and `reviewstatus_test.go:455` (`TestAgentActivity_LiveLockWaitReadsAsActivity`). Covered by card 12.
- **Card 12, `childAtWaitingRow`.**
  - Readers: the dobby wait-live reading and `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`. Covered by card 12.
- **Card 7, new wording of the board's task-recipe refusal (`boardengine/task.go:313`).**
  - Readers: `TestNewTask` only. Covered by card 7.
- **Card 7, `childRecipeSupported` and its refusal (`battencli/wire.go:533,539`).**
  - Covered by card 7.
- **Card 7, sites where a recipe is named for tapestry or darn and a sampler run now falls under the same rule.**
  - Sites found:
    - `landing-fixer-report-spec`, `loom-status-spec`, `landing-template-pr-review`, `loom-final-summary-spec`, `orch-template-role`, `loom-rubric-code-review` and `stencils.go`
    - `gateslot/doc.go`, `arbiterstop/*`, `sheddriver/doc.go`, `battenshed/*`
    - landingshed's `deps.go`, `doc.go`, `finalize.go`, `fixerrun.go`, `parentalign.go` and `handback.go`
    - battencli `wire.go:1086,1273,1482`
    - loomcli `arm.go:408`, `settledguard.go`, `subjects.go` and `drivewiring.go:129`
    - `reportengine/runs.go:24` `runRecipes`, its `doc.go:115` and `runs.go:1`, and `timingspans.go:71-82`
    - `docs/overview.md`
  - Card 7 covers all of these except the members named in the finding below.
  - Sites that stay true for sampler, so no card is needed:
    - `driftcli/*` (sampler has no Drift rows)
    - `shedengine/doc.go:59`
    - `shedrecipe/doc.go:81`
    - `burlerengine/doc.go:252` and `lenses/config.go:175`
    - `loomcli/doc.go:305`
    - `loomrecipe` `bouncerrows.go` and `finalizerow.go`, which card 7 deliberately keeps

**Findings**

1. **MEDIUM, design.** Location: card 7 (`07-sampler-vocabulary.md`) Edit list and Intent.
   - Summary: card 7's sweep of recipe lists and "tapestry or darn" prose covers only part of the set. These members are missed:
     - `internal/reportengine#runRecipes` (`reportengine/runs.go:24`), with the text that describes it (`reportengine/doc.go:115`, `runs.go:1`) and the phase table it feeds (`reportengine/timingspans.go:71-82`). This list holds only tapestry and darn, so every sampler run is silently left out of `lym` reports.
       - That breaks D2's "the recipe name reaches every recipe vocabulary".
       - The plan neither adds sampler to the list nor records leaving it out.
     - `internal/loomcli/drivewiring.go:129`. Its escalate comment says "a tapestry or darn landing merge fixer reports an escalate as a halt of its own", and a sampler run now falls under that rule too.
     - `internal/landingshed/handback.go`'s file comment (line 1). The Intent says to edit "handback.go's file comment", but no Edit target lists `internal/landingshed/handback.go`. The `EscalateHalt` glyph covers only line 26.
   - Evidence: `reportengine/runs.go:20-24`, `reportengine/doc.go:115`, `reportengine/timingspans.go:71,82`, `loomcli/drivewiring.go:129`, `landingshed/handback.go:1,26`.

Tool calls:
1. grep for test callers of `codeBudgetZero`/`rerouteReviewed`, the board refusal text, readers of `RecipeNames()` and the contract way-forward text. The pre-read had the production sites only.
2. Read `cmd/lym/driverstencil_test.go:20-60`, the enclosing test name at `landingdeps_test.go:531`, and runprofile's recipe handling. The `RecipeNames` reader and the profile loader were outside the manifest.
3. grep for "tapestry or darn" and similar recipe-list prose across internal, contracts, docs and pattern, to compare against card 7's list. The manifest had no repo-wide sweep.
4. Printed the matching lines in files card 7 does not list, to judge whether each rule now covers sampler.
5. grep for `shedrun.Recipe*` constants outside the planned packages, to find code-level recipe lists such as `reportengine.runRecipes`.
6. Read `peers.go:238-262`, `arbiterprompt.go:14-34` and `reportengine/runs.go:20-26`, to judge whether those recipe readers need a sampler entry.
