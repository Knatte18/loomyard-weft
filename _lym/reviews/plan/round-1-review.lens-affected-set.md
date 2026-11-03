```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, find what reads or asserts it and check that the cards cover that set.

**Per change: the set found, and the cards that cover it**

- **`shedrecipe` registry gains `Block`.** Readers: `TestNames`, `minimalEntries` / `TestEveryEntry_RejectsUnknownConfigKey`, `TestSimpleEntries_HappyPath` (card 2). `coverageGuardAllowedUnreachableEngines` (added on card 2, removed on card 6). `TestBuild_EveryRegisteredEngineBuilds` iterates `Names()`; `Block` builds with no config, so it passes without a card. Covered.
- **`loomrecipe.RecipeEngines` gains `Block`.** Readers: `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` and the shedrecipe coverage guard (card 6). Covered.
- **`shedrun.RecipeNames` gains `arras`.** Readers:
  - `ValidateRecipe` and its callers, `shedrun/seed.go:169` and `battencli/wire.go:1392`;
  - `TestRecipeNames`, `TestValidateRecipe` and `TestRecipes_KeySet` (card 7);
  - `cmd/lym/driverstencil_test.go:30`, which builds a forbidden token per name; harmless unless the stencil says "arras".

  Covered.
- **The row-keyed loom tables gain the new `Motif` row (`loomshed.NameMotif`).** Readers:
  - `loomshed.InterruptPolicies` and `InterruptPolicyFor`, read by `shedverbs/status.go:181`, `step.go:421`, `loop.go:354` and `loopwait.go:499`;
  - `TestInterruptPolicies_MatchAssembledRows`, which iterates only tapestry, darn and sampler (`internal/loomrecipe/interruptpolicy_meta_test.go:25`).

  **No card covers this.** See finding 1.
- **The `RecipeRow` fields and `Rows` output.** Readers: `TestRowNames_FollowEachRecipesRowOrder` compares whole structs (card 6), the loomcli checklist (card 9), and `recipeHoldsPlanReview` and settledguard, whose change is additive. Covered.
- **`HistoryEntry.Block`.** Readers: the status spec (card 1), `TestStatus_JSONRoundTrip` (card 1), the checklist and `RenderStatusHuman` (card 9). Reportengine and tokencount decode the field tolerantly. Covered.
- **`ProducerDef.SkipTo` and `Block`, and the `Routing` projection.** Readers: `GotoRequest`/`RelaunchRequest` docs and `gotoAdmitted` (card 1), `Build`/`RoutingFrom`/`TestRoutingOf_MatchesNewShedRowForRow` (card 5), loomrecipe's `Routing` (card 6). Covered.
- **The `checklistVisitsOf` re-sign.** Its only caller is `renderChecklist` (`internal/loomcli/checklist.go:213`), on card 9. Covered.
- **The `unsupportedChildRecipeRefusal` text.** Reader: `TestWire_WriteSeedRefusesANonLoomChildBeforeTouchingTheWorktree` (card 7). Covered.
- **The `recipeList` output.** Callers: `tools/tokencount/main.go:294`, `:296`, `:298` and `:301`. `main.go` and `TestRecentRuns` are on card 7. Covered.
- **The prose lists of the recipe vocabulary.** Readers:
  - `shedcli/doc.go`, `shedrun/doc.go`, `reportengine/doc.go` and `gateslot/doc.go`, plus the PR-Review stencil (card 7);
  - `loomrecipe/doc.go` (card 6);
  - `docs/overview.md:262`, `:267` and `:537`.

  **No card covers `docs/overview.md`.** See finding 2.

**Findings**

1. **MEDIUM, consistency.** Location: card 6 (`06-loom-recipes-motif-arras.md`).
   - **Summary:** card 6 adds the `Motif` row to arras but no `loomshed.InterruptPolicies` entry for it. It also leaves `TestInterruptPolicies_MatchAssembledRows`'s recipe list at tapestry, darn and sampler, so that both-direction pin never sees arras.
   - **Effect:** a supervisor finding an arras run interrupted at `Motif` gets no `interrupt_policy` key from the status, step or loop envelopes.
   - **Members missed:** `internal/loomshed#InterruptPolicies` (needs a `NameMotif` entry) and `internal/loomrecipe#TestInterruptPolicies_MatchAssembledRows` (needs `shedrun.RecipeArras` in its list).
   - **Evidence:**
     - `internal/loomshed/interruptpolicy.go:32`, the table keyed by every loom row;
     - `internal/loomshed/interruptpolicy.go:72-77`, where an empty return omits the key;
     - `internal/loomrecipe/interruptpolicy_meta_test.go:18-40`, which iterates tapestry, darn and sampler only;
     - `internal/shedverbs/status.go:181`, a consumer.
2. **LOW, consistency.** Location: cards 6 and 7.
   - **Summary:** `docs/overview.md` names the closed recipe sets that these cards change. Line 262 says loomrecipe assembles "the tapestry, darn, pawl and sampler recipes". Line 267 lists shedcli's arming table as "tapestry, darn, pawl, sampler and batten". Line 537 lists the loom recipes the `lym loom` verbs serve. Arras makes all three incomplete, and neither card lists the file.
   - **Member missed:** `docs/overview.md`.
   - **Evidence:** `docs/overview.md:262`, `:267` and `:537`.

**Tool calls**

1. `shedbuild/build_engines_test.go` header, `checklistVisitsOf` callers, and struct-reflect greps in shedengine tests: the manifest held neither the registry-wide build test nor the callers of the re-signed function.
2. Greps for recipe-name lists, `recipeList` callers and embedded-recipe enumerators: the manifest held none of the consumers of the closed vocabulary.
3. `driverstencil_test.go`, `docs/overview.md` lines, `ValidateRecipe` callers and `RecipeSampler` sites: needed to judge what the `RecipeNames` and doc consumers assert.
4. `cmd/lym/driverstencil_test.go` 20-60, the skills/specs recipe lists and the board recipe help: needed to see what the `RecipeNames` iteration forbids.
5. `interruptpolicy_meta_test.go` and `loomshed.InterruptPolicies`: a row-keyed table a new row lands in, absent from the manifest.
6. `InterruptPolicyFor` and its shedverbs consumers: needed to see the effect of a missing entry.
