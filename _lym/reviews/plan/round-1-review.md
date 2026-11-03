---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/05-shedbuild-block-build.md
    summary: Card 5 breaks TestBuild_EveryRegisteredEngineBuilds, which builds every registered engine with no block_gate, so the Block row's LookupBlockGate of an empty name is refused, and the card does not list the test
    fix: "In card 5, add `internal/shedbuild#TestBuild_EveryRegisteredEngineBuilds` under Edit after `internal/shedbuild#TestBuild_Errors`, and add an Intent line under Tests saying that test's one-row recipe sets `BlockGate: shedrecipe.BlockGateAlways` on the row whose engine is `shedrecipe.EngineBlock`, so the every-engine build still builds Block."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-shedbuild-block-row-parse.md
    summary: contracts/specs/shed-recipe-spec.md lists the recipe format's top-level and row keys and says every row carries an engine, and no card updates it for block, block_gate, profile_recipe or Expand
    fix: In card 3, add `contracts/specs/shed-recipe-spec.md` under Edit after `internal/shedbuild/doc.go`, and add an Intent line saying its Document shape paragraph gains the top-level `profile_recipe` key and the block row (only `name`, `block`, `block_gate` and `on_done`, no engine), with the row-key sentence rewritten to name engine rows and block rows. In card 4, add `contracts/specs/shed-recipe-spec.md` under Extends after `internal/shedbuild/doc.go`, and add an Intent line saying the spec gains an expansion paragraph naming `Expand`, the told resolver, the block exit and the block path.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/06-loom-recipes-motif-arras.md
    summary: Card 6 adds the Motif row to arras without an InterruptPolicies entry, and TestInterruptPolicies_MatchAssembledRows still builds only tapestry, darn and sampler, so the row-keyed table and its two-way pin never see arras
    fix: In card 6, add `internal/loomrecipe#TestInterruptPolicies_MatchAssembledRows` and `internal/loomshed#InterruptPolicies` under Edit after `internal/loomrecipe#TestRecipeKeys`. In the Intent, say `InterruptPolicies` gains `NameMotif` mapped to `shedverbs.InterruptPolicyReinvoke`, since the Motif producer returns Done at once and re-invoking it repeats nothing. Under Tests, say `TestInterruptPolicies_MatchAssembledRows` adds `shedrun.RecipeArras` to the recipes it builds.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-profile-recipe.md
    summary: Card 8's fallback for a ProfileRecipe refusal in prepareRunSettings, and its test row, guard a state Seed-Child never reaches, because WriteSeed refuses every recipe outside shedrun.RecipeNames and childRecipeSupported before writeRunSettings calls PrepareRunSettings
    fix: "In card 8's Intent, replace the sentence that begins \"When `ProfileRecipe` refuses, `prepareRunSettings` keeps `settings.Recipe`\" with one saying `prepareRunSettings` returns `ProfileRecipe`'s error unchanged as a fail-loud wiring guard. Seed-Child's `WriteSeed` (`internal/battenshed/seamchild.go`) has already refused any recipe that is not an embedded, child-supported one, so no task reaches it. Under Tests, drop the `TestPrepareRunSettings` row \"a task whose recipe names no embedded recipe and no profile writes the task's settings alone, with no refusal\"."
  - id: F5
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/07-arras-seedable.md
    summary: Two docs still list the recipes by name without arras after card 7 makes it seedable, and no card edits them - docs/overview.md lines 262, 267 and 537, and internal/battenshed/doc.go line 137, which card 7 lists only under Uses
    fix: In card 7, remove `internal/battenshed/doc.go` from Uses. Under Edit, add `internal/battenshed/doc.go` and `docs/overview.md` after `internal/battenshed#ErrUnsupportedChildRecipe`. Add an Intent line saying battenshed's doc names arras beside tapestry, darn and sampler in its escalate-halt sentence. Add another saying `docs/overview.md` names arras in the loomrecipe line, the shedcli arming-table line and the list of loom recipes the `lym loom` verbs serve, and names motif as a block source in the loomrecipe line.
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/06-loom-recipes-motif-arras.md
    summary: Two bullets of TestBlockRecipes, the arras Discussion ring shape and the New-versus-Routing row agreement, repeat assertions that the same test's row pin, TestNew_ShapeMatchesRecipe and card 5's TestRoutingOf_MatchesNewShedRowForRow already make
    fix: In card 6's Tests, delete the TestBlockRecipes bullet that asserts the built Discussion ring shape. Keep only its first clause, that `New(shedrun.RecipeArras, …)` over `testEnv` builds and so passes the loom contract check. Also delete the bullet asserting that `New`'s built rows and `Routing(shedrun.RecipeArras, …)`'s projected rows agree on Name, OnDone, OnStuck, SkipTo and Block.
  - id: F7
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/07-arras-seedable.md
    summary: Card 7's arras pass of TestWire_DefaultConfig repeats card 6's arras row of TestReviewBudget_AppliedByNewAndRouting, since loomSegmentBudgets keys budgets by segment name and arras's review segments carry tapestry's names
    fix: In card 7, remove `internal/loomcli#TestWire_DefaultConfig` from Edit and `internal/loomcli#loomSegmentPerches` from Uses, and delete the Tests bullet that begins "`TestWire_DefaultConfig` gains an arras pass of its budget block".
---

### [MEDIUM] Card 5 fails its own gate on the every-engine build test

**Location:** `_lym/plan/05-shedbuild-block-build.md`

**Issue:** `TestBuild_EveryRegisteredEngineBuilds` (`internal/shedbuild/build_engines_test.go:89`) builds a one-row recipe for each name in `shedrecipe.Names()` and sets no `BlockGate`.
Card 2 registers `Block`.
Card 5 gives a row whose engine is `Block` the gate `shedrecipe.LookupBlockGate(row.BlockGate)` builds, and `LookupBlockGate` refuses any name the table lacks, the empty name included.
From card 5 on, the test's `Block` subtest fails with `Build() for engine "Block": error`.
Card 5 does not list the test, so the card cannot pass its own gate.

### [MEDIUM] The recipe-format spec goes stale

**Location:** `_lym/plan/03-shedbuild-block-row-parse.md`, `_lym/plan/04-shedbuild-expand.md`

**Issue:** `contracts/specs/shed-recipe-spec.md` defines the recipe document's shape.
It names the optional top-level keys and states "Each row carries `name`, `engine`, `config`, `on_done`, `on_stuck`, `on_reroute`, `segment`, and `max_bounces`".
Card 3 adds the `block` and `block_gate` row keys, a row with no engine, and a top-level `profile_recipe`.
Card 4 adds expansion.
No card lists the spec, so the format's contract no longer describes the format the plan ships.

### [MEDIUM] InterruptPolicies has no entry for the new Motif row

**Location:** `_lym/plan/06-loom-recipes-motif-arras.md`

**Issue:** `loomshed.InterruptPolicies` (`internal/loomshed/interruptpolicy.go`) maps every loom row to its interrupt policy.
`shedverbs`' status, step and loop envelopes omit `interrupt_policy` for a row with no entry.
`TestInterruptPolicies_MatchAssembledRows` (`internal/loomrecipe/interruptpolicy_meta_test.go:18-40`) is the two-way pin that catches a row added with no policy, and it builds only tapestry, darn and sampler.
Card 6 ships arras, whose `Motif` row has no entry, and the pin never builds arras, so the gap passes every gate.
D10 has arras wired wherever tapestry is, and every other arras row inherits its policy by name.

### [MEDIUM] Card 8's fallback branch cannot be reached

**Location:** `_lym/plan/08-profile-recipe.md`

**Issue:** Card 8 has `prepareRunSettings` keep the task's recipe when `ProfileRecipe` refuses.
It says such a task then reaches Seed-Child's seed write, and it adds a `TestPrepareRunSettings` row for it.
That order is backwards: Seed-Child's `WriteSeed` (`internal/battenshed/seamchild.go:93`) runs before `writeRunSettings` calls `PrepareRunSettings` (`:141`).
`WriteSeed` already refuses any recipe outside `shedrun.RecipeNames` or rejected by `childRecipeSupported`.
So every recipe that reaches `prepareRunSettings` is an embedded recipe, and `ProfileRecipe` never refuses there.
The rubric's reachability rule flags a guard branch whose input an upstream check rejects; the upstream check is `WriteSeed`'s `shedrun.ValidateRecipe` and `childRecipeSupported`.

### [LOW] Docs that list recipes by name leave out arras

**Location:** `_lym/plan/07-arras-seedable.md`

**Issue:** `docs/overview.md` names the closed recipe sets in three places:
- line 262, the recipes loomrecipe assembles;
- line 267, shedcli's arming table;
- line 537, the loom recipes the `lym loom` verbs serve.

`internal/battenshed/doc.go:137` names "a tapestry, darn or sampler fixer's escalate halt".
Card 7 makes arras a seedable loom recipe with the same fixers, so each of these sentences becomes incomplete.
Card 7 lists `internal/battenshed/doc.go` only under Uses, and no card lists `docs/overview.md`.
The affected-set and repo-invariants lenses both reported this gap; it is merged here.

### [LOW] TestBlockRecipes repeats covered assertions

**Location:** `_lym/plan/06-loom-recipes-motif-arras.md`

**Issue:** TestBlockRecipes already pins arras's expanded rows equal to tapestry's, config, edges, segment and block path included.
`TestNew_ShapeMatchesRecipe` (`internal/loomrecipe/recipe_test.go:25`) already asserts the Bouncer → review → fix → Bouncer ring over tapestry's built rows.
Given both, the ring-shape bullet asserts nothing new for arras.
The New-versus-Routing bullet repeats card 5's new `TestRoutingOf_MatchesNewShedRowForRow` recipe, which compares `NewShedFrom` and `RoutingFrom` over an expanded block recipe on `SkipTo` and `Block` row for row.
`New` and `Routing` in loomrecipe both read the same `parseEmbeddedRecipe` output.

### [LOW] Card 7's arras budget pass repeats card 6's

**Location:** `_lym/plan/07-arras-seedable.md`

**Issue:** `loomSegmentBudgets` and `loomSegmentPerches` key budgets by segment name.
Arras's review segments carry tapestry's names, `Discussion-Review`, `Plan-Review` and `Code-Review`.
So `TestWire_DefaultConfig`'s tapestry budget block (`internal/loomcli/wiring_test.go:448-496`) already pins the per-segment mapping.
Card 6's arras row of `TestReviewBudget_AppliedByNewAndRouting` already pins that `New` and `Routing` apply a segment's budget to arras's inlined Discussion-Review rows.
The arras pass the card adds asserts neither anything new.

## Rejected

- guard-reach F1 (card 4 should refuse a sub-recipe whose `entry` names no row of its own recipe): D2's refusal list is closed and does not license this one. The one shipped sub-recipe, motif, has its entry pinned through TestBlockRecipes' expanded-row equality. An `entry` naming no row is already stopped by `shedengine.validate` before any run.

## Focus notes

- Focus 1: `internal/loomrecipe/recipekeys.go`'s retired rows map row names from runs that predate a rename. Arras has no older runs, and `RetiredRows`' only run-keyed caller, `loomcli/arm.go:288`, correctly gets nil for arras; the other callers are keyed to tapestry and darn explicitly. Leaving arras out is not a defect. D8's single accessor holds: `prepareRunSettings` is the one reader of a profile block by a task's recipe. `boardcli`'s check reads a profile by name only, and `configreg.checkProfiles` and `runprofile`'s defaults decoder validate every block as written.
- Focus 2 and 3: Every Expand, Build and `RoutingOf` refusal is reached by its named input. `Parse` checks no routing and admits a block row with an empty engine or `on_done`, so no earlier check intercepts. See guard-reach's list.
- Focus 5: Card 6 is one reviewable unit. The constants name the recipe files the card ships, and `embeddedRecipes` is a private table serving the accessors on the same card. Not a defect.
- Focus 6: Card order holds. Card 2's coverage allowlist entry is removed on card 6, where `RecipeEngines` first reaches `Block`. Card 7's `Uses` of `NameMotif` and card 9's of `RecipeRow` come after card 6.
- Focus 7: No `Edit` glyph whose Intent changes a signature lacks the arrow. `checklistVisitsOf` carries it, and `parseEmbeddedRecipe`, `prepareRunSettings`, `arbiterRecipeDescription` and `recipeList` keep their heads.
- Focus 8: Card 1 states the skip-target and goto bounds. The `always` gate can only enter a block, so it skips nothing.
- Focus 9: Every `Extends` target only gains content. Card 2's rewrite of the registry's doc comment sits under the `registry` member in Edit.
- Focus 10: Card 2's `PATTERN.md` text equals the declared change. No card breaks `PATTERN-sole-parsers`, `PATTERN-named-constants`, `PATTERN-tolerant-records` or `PATTERN-shed-driver`.
