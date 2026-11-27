# gatecli: run the long scenario tests' rows in parallel, a hub each

`TestGateTest_Scopes` and `TestGateTest_Scenario` each ran one long sequence of subtests over a single shared hubforge hub.
A row left config, files, commits or a green record behind and restored them for the rows after it, so the rows ran one after another.
The 2026-11-27 test-time study put them at 54 s and 45 s, the two longest tests of `internal/gatecli` after the .NET ones.

## What changes

- `scopes_integration_test.go` gains `scopesHub` and `newScopesHub`.
  A hub of its own carries the task pair, the seeded scopes module on the prime and the pair fast-forwarded onto it, plus the one-card plan.
  The old closures become methods; helpers that need no hub become package functions.
- `test_integration_test.go` gains `scenarioHub` and `newScenarioHub`.
  Each hub has the default config, a dirty prime and the shared-build directory.
  `withSharedBuild`, `buildRecords` and `closingOf` move to package level, and `buildRecords` also replaces the scopes file's copy of the same helper.
- Every row of both tests calls `t.Parallel()` and builds its own fixture.
  Rows over a standalone worktree, or outside every hub, build no hub.
  Nested tables build a fixture per nested row.
- In the refusals table, each row breaks its own hub and repairs it before checking the slots.
  Its `setup` and `wantError` now take the row's fixture.
- Every assertion stays.
  The `t.Cleanup` blocks and `defer`ed config rewrites that only put the shared hub back for later rows are gone,
  as is the removal of a green record at the start of a row, which a fresh hub never holds.

## Design decisions

- **Rows that read an earlier row's state now set it up themselves.**
  Three rows used a green record an earlier row had written:
  "a changed tier command, a different tree and -- -count=1 each break the carry",
  "an --all run runs after a committed non-inert change although a whole ancestor is green"
  and "an --all run on a dirty tree runs whole although the history is inert-only".
  Each now opens with an `--all` run of its own, asserted to run `./...` whole, before its original steps.
  "a real go test tier in the nested module" now creates the nested directory that the row before it used to create.
- **The plan is written for every scopes row.**
  It sits under the never-tracked `.lym` directory, so no row's diff sees it.
  The rows that used to run before it was written behave the same.
- **No scope case moved to tier0.**
  The only candidate that is not a whole `lym gate test` call is `TestScanGuards_DropsWatchedGuardsNoReadChangeConcerns`.
  It reaches `impactset.ModuleGuardTests`, which runs `go list`, so it stays in tier1.
  Every row of `TestGateTest_Scopes` runs the gate binary, git and `go list`, and asserts on run records and output only that process writes.
  The pure helpers those rows reach, such as `withMainPackages`, already have their tier0 test.
- `TestGateRedFirst_Dotnet` and `TestGateTest_DotnetFixture` are untouched.
- The timings below come from `lym gate test --wait --no-carry --tier 1 ./internal/gatecli -- -count=1 -v`, the study's command.
  That call takes no `--gate`, because `--gate` refuses `--tier`.
  So it also runs the `//lym:only upkeep` .NET tests, which a gate key's run skips.

## Measurements

The package's tier1 time, twice on the base commit `b65f0be65` and twice after this change, with the 1-minute load average at each start:

| Run | Load at start | Package test time (go's `ok` line) | Wall | Longest top-level tests |
|---|---|---|---|---|
| before 1 | 4.87 | 31.9 s | 39.9 s | `TestGateRedFirst_Dotnet` 31.0, `TestGateTest_Scopes` 23.6, `TestGateTest_Scenario` 21.5 |
| before 2 | 15.89 | 39.6 s | 47.4 s | `TestGateRedFirst_Dotnet` 38.9, `TestGateTest_Scopes` 32.0, `TestGateTest_Scenario` 28.7 |
| after 1 | 31.21 | 48.0 s | 61.2 s | `TestGateRedFirst_Dotnet` 47.5, `TestGateTest_DotnetFixture` 21.3, `TestGateRunner` 10.9 |
| after 2 | 46.15 | 45.9 s | 59.4 s | `TestGateRedFirst_Dotnet` 44.4, `TestGateTest_DotnetFixture` 21.4, `TestGateRedFirst` 15.4 |

All four runs passed, each discovering 343 tests: 265 passed and 78 skipped, the same before and after.
The after-runs started under six to nine times the load of the before-runs, so this table shows no package-level gain or loss.
In every run the package's time is about that of `TestGateRedFirst_Dotnet`, which a gate key's run skips.
The two split tests now report 0.0 s as parents, since Go leaves parallel subtests out of a parent's time.
Their longest row took 7.3 s and 6.2 s (Scopes) and 4.2 s and 2.9 s (Scenario) in the after-runs.
This command cannot show what a gate key's run gains, since it is the one run that skips the .NET tests.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
