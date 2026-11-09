```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: pinning test, or none**

1. Perch recorded on the first call when the rows hold a split perch, and `raddle_auto_approve` unread: pinned by `TestRaddleWrite` row `ASplitPerchIsPerchAndReadsNoKey` (`raddlewrite_test.go`, `keyReads` 0). The key-off interactive and key-on auto rows pin that a `raddle` run is unchanged.
2. `firstMode` failure when `SplitPerch` errors, `raddlewrite.go` `firstMode`: none. This is a new failure path with a distinct message. No test makes `SplitPerch` return an error.
3. `clearLastAttempt` for perch, moving the proposal aside on a return or a goto: pinned by `ReturnInPerchMode` and `GotoInPerchMode`.
4. Raddle-File files a perch-mode proposal with no approval match: pinned by `APerchRunFilesWithNoApproval`.
5. The Raddle-File perch checks, meaning `Perch` reaching the Raddle-File proposal check: none. `APerchRunFilesWithNoApproval` uses a fixture check that returns canned findings, so it would pass even if the real Raddle-File check never got `Perch` set.
6. `raddleBoardContext` setting `Perch` from the state, and its new read-state error path: none. `raddlecontext_integration_test.go` calls the function, but the diff adds no row for `Perch` true, state absent, or a state read failure.
7. The gate's `StoreCold` closure in `raddlewiring.go` storing the cold proposal in perch mode (only interactive is a no-op): none. No test names `RaddleModePerch` against that closure.
8. `raddleWriteEnv` `SplitPerch` wiring (`loomrecipe.HoldsSplitPerch(recipe)` fed with `c.recipe`): none. The seam tests only pin that the seam is non-nil.
9. `freshRaddleEntry` goto matches `shedengine.Goto` and `Relaunch` entries, and the Stuck branch keeps its `RoutedTo` test: pinned by `TestFreshRaddleEntry` rows `GotoIntoTheRow`, `GotoElsewhere` and `RaddleFileStuckRoutedElsewhere`.
10. `raddleRunFiling` (store read, created-stamp match, board read errors): none. `decision_test.go` replaces it with `decisionFake.raddleFiled`, and the only caller outside the function is the `decision.go` `preRun` closure.
11. `decision add` refusal at Raddle-File in perch mode, before and after the filing: pinned by the two `TestDecisionVerb_Refusals` rows. Both assert `way forward:` and the named steps.
12. `decision add` outside perch mode at Raddle-File still appends: pinned by the `AppendsAndCommits` row "a raddle run outside perch mode at its Raddle-File row".
13. `decision add` in perch mode at a row other than Raddle-File proceeds, or a recipe with no Raddle-File row, in `perchFilingRefusal`: none. If the `currentRow != row` or `!hasRow` guard were dropped, every perch-mode `decision add` would refuse and no row would fail.
14. `perchFilingRefusal` error paths (`raddleFileRow`, `raddlePerch` or `raddleFiled` failing, giving the "read the run's raddle filing" refusal): none.
15. `preRun` closures in `decision.go` (`raddlePerch` reading the state mode, `raddleFileRow` from `EngineRow`): none. The tests replace them with fakes.
16. `raddle-propose` posts no notice in perch mode: pinned by `APerchRunPassesWithNoWaitingNotice`. It also pins the cold store and `ProposedSession`.
17. `raddle-approve` refuses in perch mode: pinned by `PerchModeIsRefusedNamingThePerch`.
18. `RaddleTable` takes the mode: perch output is the proposal file, not interactive, and the prompt holds only that mode's text, in all three modes on first session and return: pinned by `TestRaddleTable`. The perch text's prose is exempt.
19. `raddleparser` perch outcomes: pinned by `TestCheck` rows `PerchTapestryPasses`, `PerchTasksShapeWithOnePartPasses`, `PerchProjectUnderTheRunsEntryPasses`, `PerchTasksShapeWithTwoParts`, `PerchBeam` and `PerchProjectUnderAnotherSlug`. These also pin that the default is unchanged. `ProjectUnderTheRunsEntryOutsidePerch` pins `slug-sketch` outside perch, and `PerchPartUnderTheSketchsSlug` pins that the exemption covers the project slug only.
20. `RowsHoldSplitPerch` and `HoldsSplitPerch`: pinned by `TestHoldsSplitPerch`, which has the positive fixture and the embedded recipes negative. There is no row for a non-Bouncer row naming the proposal path, or a Bouncer over another path. That is partly covered, since motif and tapestry hold Bouncers over other artifacts.
21. `loomrecipe` seam-list additions: pinned by the seam enforcement tests themselves.

**Findings**

- **MEDIUM, design.** `decision.go` `perchFilingRefusal` (diff line 104, `internal/loomcli/decision.go`). Behaviour 13: no row shows a perch-mode `decision add` at another row, or on a recipe with no Raddle-File row, still appends. The acceptance criterion says the refusal applies only from Raddle-File on. Fix: add an `AppendsAndCommits` row with `raddlePerch` true and the run at a non-Raddle-File row.
- **MEDIUM, design.** `internal/loomcli/raddlewiring.go` `raddleRunFiling`, line 73. Behaviour 10: its three outcomes have no test: no store entry, a store entry from an earlier board entry whose created stamp differs, and a matching entry returning `Filed`. The post-filing refusal's slug list rests on it. Fix: add a table test over a real or temp raddle store and board.
- **MEDIUM, design.** `internal/loomcli/raddlecontext.go` `raddleBoardContext`, line 35. Behaviour 6: `Perch` set from the state is untested. This is the one signal that makes the gate, both verbs and Raddle-File's checks apply the perch outcomes (D5, acceptance criterion 2). Fix: add a case to `raddlecontext_integration_test.go` that writes a perch-mode state and asserts `Perch` true, and one with no state asserting false.
- **LOW, design.** `internal/loomcli/raddlewiring.go` `StoreCold` closure, line 157. Behaviour 7: no test covers the closure storing in perch mode and skipping in interactive mode. Fix: add a closure test over a temp state in each mode.
- **LOW, consistency.** `internal/loomshed/raddlewrite.go` `firstMode`, line 137. Behaviour 2: the `SplitPerch` error path has no test. Fix: add a `TestRaddleWrite` row where `SplitPerch` errors, asserting `Call` fails and the state records no mode.
- **LOW, design.** `internal/loomcli/decision.go` `perchFilingRefusal`, line 104. Behaviour 14: the three read-error branches and the "read the run's raddle filing" refusal have no row. Fix: add refusal rows that make each fake closure return an error.

Behaviours 5, 8 and 15 are wiring closures covered only indirectly. I did not raise them separately, because findings 3 and 6 cover the state reads.

**Tool calls**
1. `grep` over `internal/*_test.go` and callers, for `raddleRunFiling`, `raddleBoardContext`, `raddleWriteEnv` and `RaddleModePerch`: my context held the diff's test files but not the whole of `internal/loomcli`'s tests.
