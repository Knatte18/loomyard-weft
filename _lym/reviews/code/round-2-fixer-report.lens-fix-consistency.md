1. LOW: `internal/shedrecipe/coverage_guard_test.go:2`, `:26`, `:66` and `:77` still say each consumer has its own `RecipeEngines()`.
   - Evidence: `battenrecipe.RecipeEngines` no longer exists; the batten engines come from `shedcheckkit.RecipeEngines(recipes.BattenRecipe)` (line 42).
   - The file header, the allowlist comment and two failure messages were left as they were.

No other findings.

Tool calls:
1. `grep` of `internal/fabricengine/doc.go` and `coverage_guard_test.go` for stale snapshot-reader and `RecipeEngines` wording, because the context held only the pre-edit text of the lines that moved.
