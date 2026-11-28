# internal/shedcli.names is reached by no main package

`shedcli-names-unreached` · Task · enhancement, shed · starting · Seed-IDE

## Seen

The upkeep gate at `3fb0787e3` fails `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` (`cmd/lym/testonlycode_linux_test.go`):
`internal/shedcli.names`, in `internal/shedcli/table.go`, is reached by no main package of the module.
Only `internal/shedcli/table_test.go` calls it.
Since `design-first-route` (`9acc400f8`) the unknown-recipe error in `lookup` lists `shedrun.OfferedRecipeNames()` instead.

## Idea

Remove `names` with the test of it, or give it a production caller.
The file's header comment names the `names` accessor and needs the same edit.
