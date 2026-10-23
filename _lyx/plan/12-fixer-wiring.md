# Card 12 — fixer-wiring

**Create:**
- `plan:internal/loomcli#rowEntry` -> `func rowEntry(history []shedengine.HistoryEntry, row string) int`
- `plan:internal/loomcli#fixerTestFileMatcher` -> `func fixerTestFileMatcher(recipe, planDir string) func(path string) bool`
- `plan:internal/loomcli#TestRowEntry` -> `func TestRowEntry(t *testing.T)`
- `plan:internal/loomcli#TestFixerTestFileMatcher` -> `func TestFixerTestFileMatcher(t *testing.T)`

**Edit:**
- `internal/loomcli#landingDeps`
- `internal/loomcli#loomCLI.wireLanding`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`

**Uses:**
- `internal/planparser#PlanLanguage`
- `internal/planparser#IsTestFile`
- `internal/planparser#IsAnyLanguageTestFile`
- `plan:internal/fabricengine#Fabric.LineChange`

**Intent:** Fill the merge fixer's seams card 11 added, from the values the wiring already resolved, and keep `landingDeps` free of I/O and of a new argument.
`landingDeps` fills `DecisionRecordPath` with the run's decision record, `PlanDir` with the plan directory on a loom run and "" on a darn run, `LineChange` and `TaskRemoteTip` as closures that open the task pair's fabric at call time (`TaskRemoteTip` resolves `origin/<task branch>` with no fetch and answers "" when it does not exist), `IsTestFile` with `fixerTestFileMatcher(recipe, planDir)`, and `RowEntry` with a closure that reads the run's status file under its lock and answers `rowEntry(history, row)`.
`fixerTestFileMatcher` answers `IsAnyLanguageTestFile` for the darn recipe; for loom it parses the live plan at call time through `planparser.ParsePlan` and answers `IsTestFile(PlanLanguage(plan), path)`, so a plan with no `language:` key reads as `go`, and a plan that does not parse reads as `none`, whose matcher counts every file a test file and over-covers.
`rowEntry` answers the history index just after the last entry another producer made, so every entry the row adds while it is resumed keeps the same identity, and an entry of the row reached again from another row gets a new one.
`wireLanding` sets `Landing.FixerSeats` to the seat engine `wire` already built (`c.env.Seats`), so no second engine is constructed, and the drift test names `FixerSeats` as the one field its caller fills.

`TestRowEntry` pins an empty history, a history ending in the row's own resumed entries, and one where another producer's reroute entry precedes the row.
`TestFixerTestFileMatcher` pins the darn union, a loom plan with no `language:`, a `csharp` plan and an unparseable plan directory.

**ImpactSummary:** One call site (`wireLanding`) and the assembly seam change; the new seams have no reader until Merge-In and Finalize run the fixer.
