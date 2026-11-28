# gatecli: run TestGateRedFirst_Dotnet's rows in parallel

`TestGateRedFirst_Dotnet` is a slow test in `internal/gatecli`.
Each of its three rows built the .NET solution cold in a fresh checkout of the base,
and the rows ran one after another over one shared working tree.

Each row now copies its own hub and runs in parallel, and there are two rows instead of three.
Only the test file changes.

## Design decisions

- The test that passes on the base no longer has a row of its own.
  It runs beside the failing test in the red row,
  which asserts that both are selected, that only the failing one is named red,
  and that the run reports `discovered 2 passed 1 failed 1`.
- This loosens the test: no real `dotnet` run of `lym gate red-first` now asserts the exit code
  `gateslot.RedFirstGreenExit` or the "does not show the bug" text.
  `TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase` still asserts both, with stand-in tier runs.
- Keeping that case as a third parallel row was measured and rejected.
  In alternating runs started at a 1-minute load of 34 to 52,
  the longest parallel row took 18.1 to 23.6 s with three rows against 12.5 to 14.2 s with two.

## To check by hand

- Time the whole test and the `internal/gatecli` package before and after this change if the saving matters.
  The figures above compare only the two parallel layouts, not the old serial test or the package time.

- Accept or reject the lost coverage in `TestGateRedFirst_Dotnet`
  (`internal/gatecli/redfirstdotnet_integration_test.go`).
  No row has a real `dotnet test` run that exits 0 in red-first's exported checkout,
  so a defect on that path that turns an all-passing run into a non-zero exit is not caught there.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
