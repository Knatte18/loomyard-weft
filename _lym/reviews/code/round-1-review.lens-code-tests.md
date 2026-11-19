```yaml
lens: code-tests
outcome: reported
```

I reviewed the diff's tests through the code-tests lens: for each behaviour the diff adds or changes, which test row would fail if it broke.

**Per behaviour, the test that pins it, or none**

1. `testcover.logsOnly`, a statement that is a logger call: pinned by `TestPlanLineMutants` rows `ALineOfALoggerCallAStringAndAnEmbeddedFileUseGetsNone` and `EveryChangedLine…` (lines 43 and 47 shadow and alias the logger).
2. `logsOnly`, an `if` with no else whose body only logs, beside one that also returns: pinned by row `AnIfThatOnlyLogsGetsNoneButAnIfThatAlsoReturnsKeepsItsCondition`.
3. `logsOnly`, the `else` block branch, the `else if` recursion and the `Init != nil` refusal: none.
4. `blockLogsOnly`, an empty block counting as not log-only: none.
5. `comparesOtherPlatform`, `runtime.GOOS == "<other>"` marked noise and `!= "<this>"` kept: pinned by row `AComparisonOfTheOSWithAnotherPlatformGetsNone…`.
6. `comparesOtherPlatform`, the literal on the left (`"plan9" == runtime.GOOS`), a shadowed or aliased `runtime`, and a non-`==`/`!=` op: none.
7. A noise site propagating through `PlanMutants` into `Mutant.noise` and `mutatesBehavior`: pinned indirectly by the `TestPlanLineMutants` rows above.
8. `PlanMutants` still emitting noise sites so `lym gate redundant` is unchanged, as the summary claims: none. `internal/testcover/mutant_test.go` has no logger or GOOS fixture.
9. `selectWithinCap` with `spreadEvenly`: pinned by `TestDiffMutation_SelectWithinCap`. I traced all three rows by hand and they hold. Cap 4 over 8 groups picks indices 0, 2, 4, 6, giving a {1,5} and b {1,3}. Cap 10 takes round 0 whole, then a.F0 and a.F2 in round 1.
10. `MutationReport.SummaryLine`, with counts, the errors suffix and a note: pinned by `TestMutationReport_SummaryLine`.
11. `MutationReport.ReviewLines`, `killers` and the line cap: pinned by `TestMutationReport_ReviewLines`.
12. `mutationStep.run` and `answer` on the default judged path in a task pair: pinned by `TestGateMutate/AGateCallListingMutationWritesItsFindingsAndPrintsOneSummaryLine`. It checks counts, gate key, functions, one summary line and the HEAD match.
13. `mutationStep.run` with a red gate, outside a hub or with a non-`go` scoper: none.
14. `mutationStep.answer` with no parent branch, in the prime, on an error line (with the exit code unchanged) and on cancellation: none.
15. `run.go` running the step only when `reports` lists `mutation`: covered both ways. `TestGateTest_Scopes` "--gate with scope diff…" with `reports: []` asserts only the tier invocations; the integration row above covers the listed case. Order after redundancy: none (minor).
16. `MutationReportMatches` returning true at HEAD: pinned by the integration row.
17. `MutationReportMatches` returning false for a changed tree, or for a Note, RedGate or Prime report: none for the mutation wrapper. The shared `changedFunctionsMatch` is pinned through `reach_integration_test` `AReportHoldsForHeadUntilACommitChangesWhatItNamed`.
18. `ReachReportMatches` refactored into `changedFunctionsMatch`: pinned by existing reach integration rows.
19. `runMutate` refactored into `mutationCall.run`: pinned by the existing `TestGateMutate` verb rows (counts, cap, budget, not judged, unbuildable). The cancel path is unpinned, as it was before the diff.
20. `gateslot` accepting `ReportMutation`, the refusal message for an unknown report and the template lists: pinned by `TestLoadConfig` rows `a key listing every report loads` and `an unknown report`, plus `templateDefaults`.
21. `renderMutationReportNote`: pinned for no report, red gate, prime, other tree and match by `TestRenderMutationReportNote`.
22. `renderMutationReportNote` for a non-red `Note` report (no hub, no parent, scoper) and for a `matchesHead` error: none.
23. `mutationReportNote` wiring: pinned by `TestWire_ReviewKeysReachTheEnv` (no-report text) and `TestWire_EverySeamFilled` (with envkit `nilLegal`).
24. Marker threading through `ReadRubric`, `BouncerConfig.rubricNotes` and `reviewRubricNotes`: pinned by `TestLoomRubrics`, `TestBouncerEntry_HappyPath` and `TestBurlerReviewEntry_RubricStencil/FillsTheReachAndMutationReportNotes`.

**Findings**

- **MEDIUM**, class `design`, `internal/gatecli/mutation.go:308`: the mutation step's judge-nothing branches have no test.
  - The untested branches: red gate (`RedGate`, `noteRedGate`), no hub (`noteNoHub`), non-`go` scoper (`noteScoper`) and, in `answer`, no parent branch (`noteNoParent`).
  - A broken branch here would run a full mutation check on a red gate or outside a hub, unseen.
  - The reach step pins the same branches: `internal/gatecli/reach_integration_test.go:333-342` covers red and no hub, and `:535-544` covers no parent. The mutation step has no counterpart; the only step row is `mutate_integration_test.go:491`, which is the judged path.
- **MEDIUM**, class `design`, `internal/gatecli/mutationreport.go:494`: `MutationReportMatches` is tested only for a true match.
  - Untested: false for a commit that changes a recorded function, and false for a Note, RedGate or Prime report.
  - The review note relies on that false to keep a stale report out of Code-Review. The only assertion is the HEAD match at `mutate_integration_test.go:522`.
  - Reach pins its own wrapper's false case at `reach_integration_test.go:410`.
- **LOW**, class `design`, `internal/testcover/mutant.go:268-313`: some noise rules have no row in `TestPlanLineMutants`.
  - `logsOnly`: the `else` and `else if` branches and the `Init` refusal.
  - `comparesOtherPlatform`: the reversed operand order and a shadowed `runtime` local.
  - Breaking the `default: return logsOnly(branch…)` arm or the sides loop fails no row. The fixture at `linemutant_test.go` lines 94-113 has no else, no init and no left-side literal.
- **LOW**, class `design`, `internal/testcover/mutant.go:359`: no test checks that `PlanMutants` still emits noise sites.
  - The summary claims "only the line planner drops them, so `lym gate redundant` is unchanged".
  - If the filter moved into `PlanMutants`, no test would fail: `internal/testcover/mutant_test.go` holds no logger or GOOS fixture (grep found none).
- **LOW**, class `design`, `internal/loomcli/mutationnote.go:357-365`: two branches of `renderMutationReportNote` have no row.
  - The `report.Note != ""` branch for a non-red note ("the report of … judged nothing: …").
  - The `matchesHead` error branch ("could not be matched against HEAD").
  - `TestRenderMutationReportNote` rows (`mutationnote_test.go:415-449`) cover neither.

Tool calls:
1. Grepped `internal/testcover/mutant_test.go` for logger or GOOS fixtures, and grepped the gatecli tests for the judge-nothing notes. My context lacked whether `PlanMutants` and the step branches are tested anywhere.
2. Grepped the subtests of `reach_integration_test.go`. My context lacked the reach step's counterpart rows, which I needed to name the covered sibling behaviour.
