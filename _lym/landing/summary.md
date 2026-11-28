# Run TestGateRedFirst_Dotnet's rows in parallel, each in its own hub

`TestGateRedFirst_Dotnet` set the minimum time of the `internal/gatecli` tier1 run.
Each `lym gate red-first` call under the dotnet scoper exports the base into a fresh checkout and builds the .NET solution there from cold.
So each row costs one cold restore, build and test run, and the three rows ran one after another over one shared working tree.

## Change

- Each row now copies its own hubforge hub, commits the fixture there and runs in parallel with the other rows.
- The row whose test passes on the base is folded into the red row as a second test method.
  That row asserts that only the test failing on the base is named in `red_tests`, so a real passing test still has to read as not red.
- The rows are a table with one envelope check loop.

## Design decisions

- The dropped row asserted the `gateslot.RedFirstGreenExit` exit code over a real dotnet run.
  `TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase` still pins that exit, with stand-in tier runs.
  `TestGateTest_DotnetFixture` still pins test counts read from a real vstest run.
  The commit records the drop in a `Loosened-Check:` trailer.
- No production code changes.
  Sharing one built fixture across rows would need red-first to reuse a checkout between calls, and each call exporting its own fresh base checkout is what the test pins.

## Measurements

Each figure is from one run that selected `^TestGateRedFirst_Dotnet$` over the tests marked `//lym:only upkeep`, with `TestGateTest_DotnetFixture` running beside it in the same package.

| | merge base `687d3580a` | this change |
|---|---|---|
| machine load average, 1 min, at start | 40.4 | 41.7 |
| `TestGateRedFirst_Dotnet` rows | 12.37 s, 7.34 s, 10.58 s in series | 13.57 s, 10.88 s in parallel |
| `TestGateRedFirst_Dotnet` wall time | 34.43 s | about 13.6 s, its longest row |
| `TestGateTest_DotnetFixture` | 16.51 s | 17.96 s |
| `internal/gatecli` package | 34.458 s | 18.001 s |

The package time is now set by `TestGateTest_DotnetFixture`.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
