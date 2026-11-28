# Remove shedcli's unreached names accessor

`internal/shedcli.names` returned the sorted keys of the recipe arming table, but no main package reached it.
The unknown-recipe error in `lookup` lists `shedrun.OfferedRecipeNames()` instead, so `table_test.go` was its only caller.
`TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` fails on any function that only tests call.

## Changes

- `internal/shedcli/table.go`: deletes `names` and its `sort` import.
  The file header now names `lookup` as the table's only accessor.
- `internal/shedcli/table_test.go`: `TestRecipes_KeySet` reads `slices.Sorted(maps.Keys(recipes))` itself and checks it against the same two expectations as before:
  the literal recipe list and `shedrun.RecipeNames()`.
  The file header no longer mentions `names`.

## Design decisions

- The function is deleted rather than given a production caller.
  No production code needs the full armed set:
  the one user-facing list, the unknown-recipe error, shows the offered recipes, which leave out a recipe that is armed but no longer offered.

## Merge danger

**Door:** two-way
**Blast radius:** shedcli
