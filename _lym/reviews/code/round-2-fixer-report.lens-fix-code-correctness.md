No findings.

Lens fix-code-correctness: `shedcheckkit.RecipeEngines` and `failsBareCheck` keep the bodies of the functions they replace, every caller compiles, and `TestRecipeEngines_ReportsEveryEngineNameSortedOnce` still pins the sorted, deduplicated result.

Tool calls:
1. Read `internal/testkit/enforcement_test.go` (first 110 lines), because the context lacked the kit import rules the new `shedbuild` import must satisfy.
