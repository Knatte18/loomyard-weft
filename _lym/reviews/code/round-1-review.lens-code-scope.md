```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the amended board entry's goal points and the `Added after Discussion` entry.

**Decisions**
- G1: Split `TestGateTest_Scopes` and `TestGateTest_Scenario` into independent subtests or tests, each with a fixture of its own, so they run in parallel.
- G2: Move scope computation cases that need no subprocess to tier0.
- G3: Keep every assertion; drop none.
- G4: Leave `TestGateRedFirst_Dotnet` and `TestGateTest_DotnetFixture` alone.
- G5: Measure the package's tier1 time before and after, twice each, and put both in the change description.
- A1 (Added after Discussion, parent): The writer runs `lym gate test --gate darn-write --wait --no-carry --tier 1 ./internal/gatecli -- -count=1 -v`, twice on the base and twice after, and records the load average at each start.

**Per change**
- C1: `scopesHub`, `newScopesHub` and the closures turned into methods, with `t.Parallel()` and `newScopesHub(t)` in every Scopes row (piece 001, lines 21–117 and on). Traces to G1.
- C2: `scenarioHub`, `newScenarioHub` and the methods, with `t.Parallel()` and `newScenarioHub(t)` in every Scenario row, nested rows included (piece 003, lines 204–425 and on). Traces to G1.
- C3: Package-level helpers. `withSharedBuild`, `buildRecords` and `closingOf` move into `test_integration_test.go`. In the scopes file, `noCarryTier1`, `wantInertEmpty`, `scopesSharedBuild` and `vetSummary` move out of the test body, and the duplicate `builds` gives way to `buildRecords`; `scopesSharedBuild` is a rename that avoids clashing with the package-level `withSharedBuild`. Forced by C1 and C2, since closures over the shared hub cannot outlive it.
- C4: The `t.Cleanup` restores, the `defer writeScoper` and `defer writeConfig` calls, the leading `removeGreen` and `os.Remove(greenPath)` calls, and the dirtying note edits that only kept a carry from leaking across rows are all removed. Forced by C1 and C2.
- C5: Priming `--all` runs, each asserted to run `./...`, are added to three rows ("a changed tier command…", "an --all run runs after a committed non-inert change…", "an --all run on a dirty tree…"), and the nested-module row now creates its own directory (piece 002, line 564; piece 003, lines 77 and 91; piece 004, lines 224–229). Forced by C1 and C2, and serves G3: each row rebuilds the state an earlier row used to leave.
- C6: The refusals table's `setup` and `wantError` now take `*scenarioHub`, and its comment is reworded (piece 005, lines 39–52 and 208–228). Forced by C2.
- C7: The scopes file header line "It is Tier 2: it builds a real hub…" becomes "Each row runs over a hub of its own…" (piece 001, line 10). Forced by C1, since the header would otherwise describe the old shared-hub layout.
- C8: The `standalone := standaloneWorktree` alias is dropped in favour of direct calls (piece 004, line 776). Forced by C2.
- C9: Commit `0593e8696`, which breaks two test comments at sentence boundaries. Forced by C1 and C2, since it fixes comments those commits added.

None of the changes is unrequested scope.

**Per decision**
- G1: built by C1 and C2, with C3, C4, C6, C7 and C8 forced by them.
- G2: no change builds it. The change description says no case qualifies because every Scopes row runs the gate binary, git and `go list`. The code supports that: every Scopes row calls `startGate`, which execs `os.Args[0]` (`test_integration_test.go:399`). The one named candidate, `TestScanGuards_DropsWatchedGuardsNoReadChangeConcerns` (`scopes_integration_test.go:1355`), runs `go list` through `impactset.ModuleGuardTests`. Since no case meets the decision's condition, I raise no finding.
- G3: served by C5. Whether the assertions are preserved one by one belongs to the other lenses.
- G4: met. The diff touches only `scopes_integration_test.go` and `test_integration_test.go` (diff stat), and the .NET tests live in `redfirstdotnet_integration_test.go` and `dotnetfixture_integration_test.go`.
- G5: built in `_lym/landing/summary.md`, `## Measurements`: two runs before, two after, each with its load at start.
- A1: built with a different command (F1).

**Findings**
- **F1** — severity LOW, class `decision`, location `_lym/landing/summary.md`, `## Design decisions` (the timings bullet).
  - Summary: the measurements were not taken with A1's command. The writer dropped `--gate darn-write` and ran `lym gate test --wait --no-carry --tier 1 ./internal/gatecli -- -count=1 -v`.
  - Why: A1's command cannot run, because `--gate` refuses `--tier` (`internal/gatecli/run.go:414-416`; `cli.go:73`).
  - Effect: the substitute also runs the `//lym:only upkeep` .NET tests, so `TestGateRedFirst_Dotnet` dominates every package time and the table cannot show what the split saves.
  - No parent ruling on the substitute is recorded, and the diff cannot fix this.
  - Fix: leave the code in place and defer to the run's parent. It can accept the substitute, or name a runnable command, for example a `-run` filter for the two split tests, as a board note for a new task.

Tool calls: none; the inherited context covered every hop this lens needed.
