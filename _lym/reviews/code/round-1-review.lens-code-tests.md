```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. `Await` reads the strand before the timeout: `TestAwait/DiedIsDecidedBeforeTimeout`.
2. `goneStrandReason` returns interrupted for a run record marked interrupted: `TestAwait/GoneStrandOfAnInterruptedRunIsInterrupted`, plus `InterruptedRunAStopOutlastingTheTimeoutEndedIsInterrupted`.
3. `goneStrandReason` returns died for a missing or unreadable run record: none. `GoneStrandIsDied` only covers a readable record that is not interrupted.
4. `fixerOutcome.note` gives its own text for an interrupted fixer: `TestWrapRowGate/AnInterruptionPastTheCapStandsWithItsNote`.
5. `rowStore.budget` leaves interrupted records out of the spent count: `TestWrapRowGate/InterruptedRecordsSpendNoBudget`, `TestRowStore`, and the budget assertion in `AnInterruptedFixerIsReplacedOnItsOwnFiles`.
6. The fixer entry replaces an interrupted fixer on the same report and reply files, for both its own spawn and a left-live record: `TestWrapRowGate/AnInterruptedFixerIsReplacedOnItsOwnFiles` (both rows call `entry.Gate()`).
7. The replacement bound of attempts plus one interrupted records: `AnInterruptionPastTheCapStandsWithItsNote`, at attempts 0 only.
8. A replacement that cannot be spawned leaves the interruption standing: none.
9. A failing replacement `Spec` build returns an error after the record was marked terminal: none.
10. `markSpawned` on the replacement, so `Expired` stays false for it: none. These are mutation survivors `rowgate.go:291` and `rowgate.go:292`.
11. `waitOnly` and `Final` (replace=false) meeting an interrupted fixer: they do not replace it, mark it terminal as interrupted and carry the interrupted note. None.
12. `judgeDiedError`, not-started with a start error, and died after starting: `TestBouncer_JudgeCall_ADiedJudgeIsAnInfrastructureFailure`, rows `NeverStarted` and `DiedAfterStarting`.
13. `judgeDiedError`, not-started with an empty `StartError`: none.
14. `judgeCall`'s died branch returns the cancellation error under a cancelled ctx: none.
15. A timed-out judge with nothing written still degrades: `TestBouncer_JudgeCall_Degradations` (the timeout row) and `Harvest/Contrast_NothingWrittenDegrades`.
16. `holdsOnlyAPIErrorPrompt` in `InputBoxText`: `TestInputBoxText`, rows "the prompt Claude Code leaves after an API error" and "a draft that only opens with the API error prompt".
17. The same prompt in `IdleSession`: `TestIdleSession`, row "the prompt Claude Code leaves after an API error".
18. The API-error prompt wrapped over two box lines still matches: none. The doc comment claims it.
19. `boxCapture` reads the plain capture when the styled one loses the box, and logs it: `TestRunner_SessionIdle_ReturnsScriptedClassification`, row "a styled capture that loses the box reads the plain capture's box".
20. `boxCapture` holds the session when both captures show no box: same test, row "a styled capture and a plain one that both show no box are held".
21. `boxCapture` keeps the styled reading when the plain capture fails: none.
22. `boxCapture`, a placeholder read from the plain capture holds the send as a draft: none. The doc comment claims it.
23. `boxCapture`, the number of styled tail lines in the log: none. This is mutation survivor `readiness.go:94`, a log field only, so not a finding.
24. `planPaneOOMNotices` gives one notice per cgroup: `TestPlanPaneOOMNotices`, through the pid 7 strand added to row `ANewKillIsToldFromTheCountToldBeforeAndAGonePaneIsForgotten`.
25. The warning a launch logs when the pane cannot be capped: `TestLaunchStrandLocked_APaneThatCannotBeCappedStillLaunchesWithAWarning/TheUserManagerCannotCapThePane`.
26. The OOM notice reaches the parent orch's creel mailbox: `TestReportPaneOOMKills_DeliversTheNoticeToTheParentOrchsCreelMailbox`.

**Findings**

- MEDIUM, design. `internal/gatefixer/rowgate.go:159` (`waitOnly` passes replace=false) and `:126` (`Final`). Behaviour 11 is new and no test pins it: no `TestWrapRowGate` row drives `waitOnly` or `entry.Final()` over a fixer that ends interrupted. Such a row would check that nothing is replaced, that the record turns terminal as `interrupted`, and that the findings carry the "no replacement ran" note. Evidence: `rowgate_test.go:556` and `:589`, the only rows that set `interruptedFirst`, both call `entry.Gate()`.
- LOW, design. `internal/gatefixer/rowgate.go:291-292`. The `markSpawned` call for the replacement survives both mutants (negated and dropped). No row checks `Expired()` while a replacement is waited on, or after an inner error leaves the replacement live. A wrong `true` there makes the writer's gate finalize early (`wait.go:416`). Evidence: the mutation report, and `rowgate_test.go:852-868`, which checks `Expired()` only for the first spawn.
- LOW, design. `internal/gatefixer/rowgate.go:287-290`. Behaviour 8 has no test: a replacement whose `SpawnOrAttach` fails should leave the interrupted outcome standing, and no row sets `spawnErr` after an interruption. Evidence: `rowgate_test.go:496`, the only `spawnErr` row, which fails on the first spawn.
- LOW, design. `internal/gatefixer/await.go:66-71`. `goneStrandReason` returns died when `FindRun` errs, and no `TestAwait` row has a gone strand with its run record missing. Evidence: the diff's await_test rows at `await_test.go:115-157`.
- LOW, design. `internal/shuttleengine/readiness.go:1040-1043`. When the plain capture fails, `boxCapture` keeps the styled reading, and no row covers that. Evidence: the two new rows at `session_test.go:358-364`.
- NIT, design. `internal/shuttleengine/claudeengine/session.go:933-942`. The doc comment says the prompt wrapped over lines still matches, and no `TestInputBoxText` row wraps it. Evidence: `session_test.go:968-969`.
- NIT, design. `internal/shedadapters/bouncer.go:779-780`. The `NotStarted` branch with an empty `StartError` has no row. Evidence: `bouncer_judge_test.go:862-863`.

No loosened check, no redundant test and no tautological test was found. The `SpawnErrorLeavesTheInnerFailureStandingWithNoRecord` assertion changed from `count()` to `spent+interrupted` but checks the same thing. Moving `Contrast_NothingWrittenDegrades` from a died run to a timed-out one follows the behaviour change, and the new infrastructure-failure test now pins the died case.

Tool calls:
1. Read the rows of `oomnotice_test.go` and the subtest names in `rowgate_test.go`. My context held only those files' diff hunks, not the existing rows and assertions I had to match each behaviour against.
