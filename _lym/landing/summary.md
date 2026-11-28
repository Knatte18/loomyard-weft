# Remove shedcli's unreached names accessor

`internal/shedcli.names` was reached by no main package of the module; only `table_test.go` called it.
`TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` in `cmd/lym` fails on such a function.
This change removes it, along with the `sort` import it alone used in `table.go`.
There is no user-visible change: the unknown-recipe error from `lookup` already lists `shedrun.OfferedRecipeNames()`.

`names` is removed rather than given a production caller.

`TestRecipes_KeySet` still pins the set of recipe names, now by sorting the keys of `recipes` itself.
The header comments of `table.go` and `table_test.go` no longer mention the accessor.

## Merge danger

**Door:** two-way
**Blast radius:** shedcli
