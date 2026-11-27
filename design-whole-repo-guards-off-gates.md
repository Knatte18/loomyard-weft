# Slow whole-repo guards, .NET builds and testmap tests leave the task gates for landing or periodic upkeep

`whole-repo-guards-off-gates` · Note · enhancement, gate, testkit

## Seen (2026-11-26)

`cmd/lym`'s test binary reaches about 2 GB and is the binary the gate's memory guard kills first when the machine is short.
Its tests build few hubs (7 `CopyHub` calls); most are guard tests that read the whole repository's source.
The heaviest is the tier1 test `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` in `cmd/lym/testonlycode_linux_test.go`:
it loads every package with `packages.LoadAllSyntax` and builds an SSA call graph of the whole program to find code only tests reach.
`cmd/lym` imports nearly every package, so nearly every change puts it in the gate's impacted set, and the whole-repo guards run in every task gate.

## Seen (2026-11-27)

In a tier1 landing gate of `pr-modes-no-agent`, `internal/gatecli` spent about 4.5 minutes of package time in two groups of tests:

- Tests that build a real .NET project: `TestGateRedFirst_Dotnet` (56 s) and `TestGateTest_DotnetFixture` (29 s).
- Tests of the coverage-map tools: `TestGateTestMap` (65 s), `TestGateMutate` (42 s), `TestGateReach` (31 s), `TestGateRedundant` (28 s), `TestRedundancyStep` (28 s).
  The report steps those tools feed are now off in every gate key of the loomyard hub (operator, 2026-11-27); the tools stay for manual use.

## Changed since (`a69580709`, `card-gate-scope-gaps`)

A card scope now holds the cards' packages and every package whose build or tests reach one of them.
So the card and batch gates also run `cmd/lym`'s listed-tier tests whenever it depends on a card's packages, which is often.
Under `--card`, the guard step still runs a `//lym:guard` test inside the scope that no listed tier covers.
Moving guards off the task gates now has the widened card scope to deal with too.

## Idea (operator, 2026-11-26 and 2026-11-27)

- Whole-repo guards that catch a break, such as tree-sitter linking and the cross-compile check, leave the task gates and run once at landing.
- An upkeep run takes the checks that need not stop a task:
  dobby already spawns the board-tidy agent every `board_tidy_every` landings (`internal/dobbyengine`), and the upkeep run goes with it, on the main branch.
  - The dead-code check, the call-graph test above; each dead function it finds becomes a note on the board.
    Dead code breaks nothing, so finding it a few landings late costs little.
  - The .NET tests and the coverage-map tool tests above; a failure becomes a bug note.
- Find every whole-repo guard in `cmd/lym` and name for each where it runs: at landing or in the upkeep run.
