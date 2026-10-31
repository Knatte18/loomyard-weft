```yaml
lens: fix-claims
outcome: reported
```

Task: fix-claims lens. I checked every sentence the fix added or rewrote against the code at HEAD.

**LOW**: Card 5 Intent says "`RecipeEngines` parses it with the other loom recipes, its doc naming `recipes.SamplerRecipe` among its sources". The card says nothing about the next doc line.
- `internal/loomrecipe/names.go:16` reads "All three recipes are the loom module's, so the set is their union."
- That count is false once a fourth source is added.
- The card's Intent says only that the doc names the new source, so an implementer may leave the "three" sentence unchanged.
- Fix: the sentence should say the doc's source list and its count both change.

The other added or rewritten claims hold:
- **Card 5, coverage guard:** `TestCoverageGuard` exists at `internal/shedrecipe/coverage_guard_test.go:51`. `RecipeEngines` iterates a fixed source slice (`names.go:426`). `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` exists at `internal/loomrecipe/recipe_test.go:213`. Its `want` list is literal plus `wantProducerTable`, so it needs `SamplerSession` added, as card 5 says.
- **Card 5, other engines:** every other engine in the sampler rows (Preflight, RecordWrite, Bouncer and Burler, Describe, PRReview, Finalize, FrictionReflect) already comes from tapestry or darn. So `SamplerSession` is the only new engine.
- **Card 5, `recipes.go`:** it holds one `//go:embed` var per recipe, so "embeds it as `SamplerRecipe`" fits.
- **Card 5, recipe directory:** no test globs the recipe files under `contracts/recipes`, so adding the yaml at card 5 trips no directory scan.
- **Card 6:** the claim that the recipe file is card 5's agrees with card 5. The `Rows` and `RowNames` doc claim matches `names.go:453,467`.
- **Card 8:** the claim about the `Deps` field docs matches what `deps.go:480-487` would need.
- **Card 12:** the template comment claim agrees with the card's own Bound sentence.
- **Cards 3 and 9:** the rewritten text asserts nothing new about the code.

Tool calls:
1. Listed `contracts/recipes` and grepped for recipe-file scans in its tests, and checked that the two named tests exist. My context lacked the directory contents and those test locations.
2. Read `contracts/recipes/recipes.go` and `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`, and listed the tests that import the recipes package. My context lacked the embed layout and the test's `want` construction.
