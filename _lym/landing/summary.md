# gatecli: run the long scenario tests' rows in parallel, a hub each

`TestGateTest_Scopes` and `TestGateTest_Scenario` each ran one long sequence of subtests over a single shared hubforge hub.
A row left config, files, commits or a green record behind, so the rows ran one after another.
`docs/research/speed/test-time-measure-2026-11-27/report.md` measured them at 54 s and 45 s,
the two long sequences in `internal/gatecli`'s test time.
This change gives each row its own fixture so the rows run in parallel.
It touches test files only.

## What changes

- `scopes_integration_test.go` gains `scopesHub` and `newScopesHub`.
  Each hub has the task pair, the scopes module seeded on the prime, the pair fast-forwarded onto it, and the one-card plan.
- `test_integration_test.go` gains `scenarioHub` and `newScenarioHub`.
  Each hub has the default config, a dirty prime and the shared-build directory.
- The old closures become methods on those types, or package functions where they need no hub.
  `buildRecords` also replaces the scopes file's copy of the same helper.
- Every row of both tests calls `t.Parallel()` and builds its own fixture, as does every row of their nested tables.
  Rows over a standalone worktree, or outside every hub, build no hub.
- In the refusals table, each row has a hub of its own, and a row that breaks it repairs it before the slots are checked.
  Its `setup` and `wantError` now take the row's fixture.
- Every assertion stays.
  The cleanups and config rewrites that only restored the shared hub for later rows are gone.

## Design decisions

- **Rows that read an earlier row's state now set it up themselves.**
  These rows used a green record an earlier row had written, and now open with an `--all` run of their own,
  asserted to run `./...` whole:
  - "a changed tier command, a different tree and -- -count=1 each break the carry";
  - "an --all run runs after a committed non-inert change although a whole ancestor is green";
  - "an --all run on a dirty tree runs whole although the history is inert-only".
- "a real go test tier in the nested module passes with nothing discovered" now creates the nested directory itself.
- "a dirty worktree diffs from the parent merge base although the record holds a whole-green ancestor"
  now commits a change before its `--all` run, so that ancestor is not the merge base.
- **The card plan is written for every scopes row.**
  It sits under the pair's plan directory, which its code repository excludes, so no row's diff sees it.
- **No scope case moved to tier0.**
  Every row of `TestGateTest_Scopes` spawns processes.
  The nearby `TestScanGuards_DropsWatchedGuardsNoReadChangeConcerns` stays in tier1:
  it reaches `impactset.ModuleGuardTests`, which runs `go list`.
- `TestGateRedFirst_Dotnet` and `TestGateTest_DotnetFixture` are untouched.

## Measurements

The package's tier1 time, measured with `lym gate test --wait --no-carry --tier 1 ./internal/gatecli -- -count=1 -v`.
This command also runs the `//lym:only upkeep` .NET tests.

| Run | Commit measured | Load (1 min) at start | Package time (go's `ok` line) | Wall | Longest top-level tests (s) |
|---|---|---|---|---|---|
| before 1 | `b65f0be65` (base) | 4.87 | 31.9 s | 39.9 s | `TestGateRedFirst_Dotnet` 31.0, `TestGateTest_Scopes` 23.6, `TestGateTest_Scenario` 21.5 |
| before 2 | `b65f0be65` (base) | 15.89 | 39.6 s | 47.4 s | `TestGateRedFirst_Dotnet` 38.9, `TestGateTest_Scopes` 32.0, `TestGateTest_Scenario` 28.7 |
| after 1 | `49eb037c5` | 31.21 | 48.0 s | 61.2 s | `TestGateRedFirst_Dotnet` 47.5, `TestGateTest_DotnetFixture` 21.3, `TestGateRunner` 10.9 |
| after 2 | `49eb037c5` | 46.15 | 45.9 s | 59.4 s | `TestGateRedFirst_Dotnet` 44.4, `TestGateTest_DotnetFixture` 21.4, `TestGateRedFirst` 15.4 |

- Both after runs predate `0593e8696`, `ef9ddf931`, `1cf73dbd9` and `b48c059f2`.
  Of these, `ef9ddf931` changes what a test does (the dirty-worktree row's added commit), and the package was not timed again.
- All four runs passed, each with 343 tests discovered, 265 passed and 78 skipped.
- The after runs started at loads of 31 and 46, against 5 and 16 for the before runs,
  so the table does not show a package-level gain or loss.
- In every run the package's time is close to that of `TestGateRedFirst_Dotnet`, an upkeep-only test.
  This command therefore does not show what a run that skips the .NET tests gains.
- In the after runs the two split tests report about 0 s as parents, since Go leaves parallel subtests out of a parent's time.
  Their longest rows took 7.3 s and 6.2 s (Scopes) and 4.2 s and 2.9 s (Scenario).

## To check by hand

- Time the package again at the branch head, on a machine loaded like the before runs,
  with the .NET upkeep tests left out, to see what the split gains.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
