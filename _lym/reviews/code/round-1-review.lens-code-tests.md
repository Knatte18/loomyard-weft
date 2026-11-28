```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. An attempt strand spawns with `weaver.attempt_timeout_min` as its timeout (`run.go` runBatch): pinned by `TestRun` (first subtest), whose `spec.Timeout != 45*time.Minute` check is at `run_test.go:394`.
2. A crashed attempt is probed and attached with the same bound (`resume.go:163`): pinned by `TestRun_Resume`, through `probed[0].Timeout != 45*time.Minute` at `resume_test.go:156`.
3. The default of 90 and the refusal of a non-positive value naming the key (`config.go`): pinned by `TestLoadConfig`, in `templateDefaults` and the row "a zero attempt_timeout_min names the key" (`config_test.go:67,81`).
4. The summary's per-batch deviations and departures lines, `none` when empty, files in backticks, departures joined with `; ` (`verify.go` writeSummary): pinned by the two rows of `SummaryNamesEachBatchsDeviationsAndDepartures_*` (`run_test.go:418-445`). Only one departure is used, so the `; ` join is unpinned. That is a NIT and is not raised.
5. One gate span per run of the verify, the rerun included (`verify.go` timedVerify, runVerify): the kinds are pinned by `TestRun` first subtest (`run_test.go:399`), by the fixer-clears test (`run_test.go:546`) and by the exhausted test (`run_test.go:558`).
6. One rework span per fixer, from its spawn to its end (`run.go:744-746`): the kind and its order are pinned at `run_test.go:546,558`. The start and end instants are pinned by none (finding 1).
7. A call that never reaches the verify carries nil spans: pinned at `run_test.go:454` (the run stopped on the last rung).
8. A stuck result from an exhausted fixer ladder carries the spans (the defer in finishRun): pinned at `run_test.go:558`.
9. A stuck result from finishRun's other returns (a runVerify error, an unjudged verify, a fixer error, a save error) carries the spans through the same defer: none. The exhausted test covers the defer itself, so no finding is raised.
10. A fixer that a crashed call left running, attached by `resumeFixers`, adds no span (a claim in the change description): none (finding 2).
11. The producer maps `VerifySpans` onto the row's `GateSpans` as entry `verify`, on done and on stuck (`shedadapters/weaver.go`): pinned by `DonePointsAtTheSummaryAndCarriesTheVerifySpans` and `StuckReasonPassesThroughWithTheVerifySpans...`. Both compare against literal RFC3339 strings (`weaver_test.go:80-115`).
12. The producer gives no gate spans for a run with no verify: pinned by `RunWithNoVerifyCarriesNoGateSpans` (`weaver_test.go:118-126`).
13. Board point 4, the paths the bench did not reach:
    - Third rung: `FailedAttemptsClimbToTheThirdRungWhichRecordsTheBatchDone` checks the models a, b, c, the third prompt's failure digest and attempt 3 done on rung 2. The prompt string matches `run.go:428` and `strand.go:258`.
    - Lost strand: the `LostStrandRetriesOnceOnTheSameRungThenClimbs_*` rows. Their causes match `strand.go:233,237`, so `paneDies` reaches the died-before-report classification.
    - Resume: the rows `Crash_FailedSecondRungAttemptRunsTheThirdRung` and `Crash_LostLastRungAttemptRetriesOnTheLastRung` (`resume_test.go:279-294`).
    - Failed verify: covered by items 5, 6 and 8.
14. Loosened checks: the old `MachineryLossRetriesOnceOnTheSameRungThenClimbs` became the `WaitError` row. That row keeps the models assertion, checks every attempt's number rather than only attempt 2's, and adds a cause check, so nothing is loosened.

**Findings**

1. MEDIUM, design. `internal/weaverengine/run_test.go:399,546,558` (`spanKinds`).
   - The verify span tests assert only the span kinds, never their Start and End instants.
   - A timedVerify or finishRun that recorded the wrong interval would pass every test: swapped start and end, a fixer span started after `runFixer`, or zero times.
   - The fixture clock never advances: `fakeClock` moves only on Sleep and starts at `time.Unix(0, 0)` (`run_fixture_test.go:44-56,628`). So every span is measured at one instant, and the measured intervals that board point 3 is about are uncovered.
   - Fix: have the verify and fixer fakes advance the fixture clock, then assert each span's start and end against literal instants.
2. LOW, design. `internal/weaverengine/resume_test.go` (the live-fixer attach rows near `:240`).
   - The change description says a fixer that a crashed call left running, attached by `resumeFixers` (`resume.go:217-266`), adds no span.
   - No resume row asserts `result.VerifySpans` after such an attach, so a change that timed the attach would go unnoticed.

Tool calls:
1. Grepped the cause and failure-digest strings in `weaverengine`, the fixture's Clock, and read `resume_test.go:240-330`. The manifest held none of these: the strings the new tests assert, whether the fixture clock moves, and the existing resume rows to compare for duplication.
