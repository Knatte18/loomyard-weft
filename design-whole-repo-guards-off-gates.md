# Whole-repo guard tests run once at landing, and the dead-code check seldom, not in every task gate

`whole-repo-guards-off-gates` · Note · enhancement, gate, testkit

## Seen (2026-11-26)

`cmd/lym`'s test binary reaches about 2 GB and is the binary the gate's memory guard kills first when the machine is short.
Its tests build few hubs (7 `CopyHub` calls); most are guard tests that read the whole repository's source.
The heaviest is the tier1 test `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` in `cmd/lym/testonlycode_linux_test.go`:
it loads every package with `packages.LoadAllSyntax` and builds an SSA call graph of the whole program to find code only tests reach.
`cmd/lym` imports nearly every package, so nearly every change puts it in the gate's impacted set, and the whole-repo guards run in every task gate.

## Idea (operator, 2026-11-26)

- Whole-repo guards, such as tree-sitter linking and the cross-compile check, leave the task gates and run once at landing.
- The dead-code check, the call-graph test above, runs seldom, not on every landing: dead code breaks nothing, so finding it a while later costs little.
- Find every whole-repo guard in `cmd/lym` and name for each where it runs.
