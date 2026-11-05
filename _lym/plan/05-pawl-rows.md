# Card 5 — pawl-rows

**Edit:**
- `contracts/recipes/pawl-recipe.yaml`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe/doc.go`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#validPawlDeps`
- `internal/shedrecipe#TestNames`
- `internal/landingshed#TestBeamDescribe`
- `internal/landingshed/beamdescribe.go`
- `internal/landingshed/doc.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#pawlDeps`
- `internal/loomcli/doc.go`
- `internal/loomcli#intentionallyNil`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#intentionallyNil`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/loomrecipe#wantPawlRows`
- `internal/loomrecipe#TestPawlRecipe`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `internal/loomrecipe/doc.go`
- `internal/battencli#runTakesNoSlot`
- `internal/pawlengine#Deps`
- `internal/pawlengine#BeamAwait.interruptedLanding`
- `internal/pawlengine#awaitRun.accept`
- `internal/pawlengine#BeamAwait.Call`
- `internal/pawlengine/doc.go`
- `internal/pawlengine_test#TestBeamAwait`
- `internal/pawlengine_test#world.deps`
- `internal/boardengine#holdFreeProducers`
- `internal/boardengine#RunHolds`
- `internal/boardengine/doc.go`
- `internal/beamcli#landCmd`
- `internal/beamcli/doc.go`

**Delete:**
- `internal/shedrecipe#beamDescribeEntry`
- `internal/pawlengine#parentAlignName`
- `internal/loomrecipe#TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign`

**Create:**
- `plan:internal/pawlengine#finalizeName` -> `const finalizeName = "Finalize"`
- `plan:internal/loomrecipe#RetiredRowNames` -> `func RetiredRowNames(recipe string) ([]string, error)`
- `plan:internal/loomrecipe#TestPawlRecipe_HoldLiftsOnlyBeforeFinalize` -> `func TestPawlRecipe_HoldLiftsOnlyBeforeFinalize(t *testing.T)`

**Uses:**
- `internal/landingshed#WriteBeamDescription`
- `internal/loomrecipe#RowNames`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`

**Intent:** Pawl runs Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down (D1), and its landing hold starts at Finalize (D2).

- `pawl-recipe.yaml`: Beam-Await's `on_done` is Finalize; the Parent-Align, Beam-Describe and PR-Review rows go, PR-Review's `on_reroute` with them (D10); a `retired:` map, parsed by card 1, maps Parent-Align, Beam-Describe and PR-Review each to Beam-Await (D13). The header comment says the hold boundary is Finalize, that the orch writes the description and handles the pull request at Beam-Await, that Finalize merges the parent in and lands, and why the three rows are retired.
- `BeamDescribe` leaves `shedrecipe`'s registry with `beamDescribeEntry`, and `Env.BeamDescribe` goes; `NewBeamDescribe`, the `beamDescribeProducer` type with its `Call` and its `ShedProducer` assertion go from `internal/landingshed/beamdescribe.go`, leaving `BeamDescribeDeps` and `WriteBeamDescription` for the verb (card 12). `ParentAlign` and `PRReview` stay registered for the other recipes. `loomCLI.wire` no longer fills `Env.BeamDescribe` nor reads the beam's parent branch for it; `internal/loomcli/beamwiring.go` stays until card 12 replaces it.
- `pawlengine`: `parentAlignName` is replaced by `finalizeName`; `awaitRun.accept` writes the status naming Finalize as the landing hold. `Deps` gains `RetiredRows []string`, the recipe's retired row names, told by the wiring; nil reads as none. `BeamAwait.interruptedLanding` reads the hold by the Finalize name or any name in `RetiredRows`, so a crash under the old binary is caught (D11's entry check, extended by card 10). `loomrecipe.RetiredRowNames` returns the sorted keys of the named recipe's `retired` map, and `pawlDeps` fills `RetiredRows` from `loomrecipe.RetiredRowNames(shedrun.RecipePawl)`. `RetiredRowNames`' doc says it reads the recipe's `retired` map and never `RetiredRows`.
- The two retired-row mechanisms stay apart (card 1 states the split): the existing arm-time rename, `loomrecipe.RetiredRows` applied by `loomCLI.renameRetiredRows` through `shedengine.RenameProducers` (`internal/loomcli/arm.go`), renames a row that lives on under a new name, at arm, with no side effect; the recipe's `retired` map moves a run off a row the recipe dropped, at step, pausing it with the recipe wiring's side effects. `internal/loomrecipe/doc.go`'s `RetiredRows` paragraph and `internal/loomcli/doc.go`'s retired-row sentence say the two apart.
- `internal/loomrecipe/doc.go`'s pawl paragraph names the six rows, drops the reroute edge and `lym beam reopen`, and says a run at Finalize or any row after it holds its beam. `runTakesNoSlot`'s doc (`internal/battencli/dobbyreads.go`) says a pawl run takes a slot from Finalize on.
- `boardengine`: `holdFreeProducers` keeps Worktree-Create, Preflight, Record-Write, Beam-Up and Beam-Await; its comment and `RunHolds`' doc say the pawl run holds its beam from Finalize on.
- `lym beam land`'s help no longer says the landing writes the description: Finalize lands what the orch described.
- The seam tests drop the `//lym:seam internal/landingshed#BeamDescribeDeps` marker and the `BeamDescribe.*` entries of `intentionallyNil`, since `Env` no longer holds it; `FullEnv` drops `BeamDescribe` and fills `Pawl.RetiredRows`.

Tests (tier 0): `wantPawlRows` and `TestPawlRecipe` pin the six rows and their edges; `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` drops `BeamDescribe`; the new `TestPawlRecipe_HoldLiftsOnlyBeforeFinalize` replaces the Parent-Align one, pinning that every pawl row before Finalize holds nothing and Finalize and Beam-Down hold the beam, and that `RetiredRowNames(pawl)` is Beam-Describe, Parent-Align, PR-Review. `TestNames`, `minimalEntries` and `validPawlDeps` drop `BeamDescribe`. `TestBeamAwait`'s accept rows assert the status names Finalize; the row that enters with no land request and the status naming Parent-Align, the retired hold an old binary's accept wrote, now reaches the interrupted-landing notice through `RetiredRows`, which `world.deps` fills. `TestBeamDescribe` drops its producer row.

**ImpactSummary:** the pawl recipe loses three rows and the registry one engine; every caller of the removed producer and `Env` field is a wiring, a seam test or a fixture listed here; tapestry, darn and sampler keep Parent-Align, PR-Review and Finalize unchanged.

**Commit:** 5: Pawl lands from Beam-Await through Finalize
