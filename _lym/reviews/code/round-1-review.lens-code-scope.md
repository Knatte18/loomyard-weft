```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced each change in the diff to the goal point it builds, and checked each goal point for a change that builds it.

**Decisions.** The answer key is the decision record alone. Its `## Goal` has four points and the record has no `Added after Discussion` entries.
- **G1:** a Weaver attempt gets a timeout of its own, sized to a batch.
- **G2:** `summary.md` names each batch's deviations and departures, as Webster's summary does.
- **G3:** the Weaver row records `gate_spans` for the plan-level verify, as the Webster rows do.
- **G4:** tests pin a lost strand, a resume, a failed plan-level verify and the ladder's third rung.

**Per change:**
- **C1, the `weaver.attempt_timeout_min` key** → G1. This covers `config.go` (the `AttemptTimeoutMin` field, its category and its `LoadConfig` refusal), `keynames.go` (`attemptTimeoutMinKey`), `template.yaml` (default 90), the `doc.go` key list and the `config_test.go` default and zero row.
  - The reworded `LoadConfig` doc comment, which now cites the full `weaver.` key names, is forced by C1: the comment lists the refusals and C1 adds one.
- **C2, the attempt strand's bound** → G1. This covers `strand.go` (`attemptLaunch.Timeout` passed into `attemptSpec`), `verify.go` (`RunDeps.attemptTimeout`), `run.go` (`runBatch` sets it) and `resume.go` (`resumeAttempt` sets it on the probe and attach spec).
  - The timeout assertions in the `run_test.go` happy path, the `resume_test.go` probe assertion and the fixture's `AttemptTimeoutMin: 45` are forced by C2.
- **C3, deviations and departures in the summary** → G2. This covers `verify.go` `writeSummary` (two lines under each batch, `none` when empty) and the `doc.go` sentence.
  - The fixture's `fakeGit.commit` departures, `attemptScript.departure`, `fakeIndex` and `fakeDelta` `outsideTargets`, and the `SummaryNamesEachBatchsDeviationsAndDepartures_*` rows are forced test support.
- **C4, verify gate spans** → G3. This covers:
  - in `run.go`: the `RunResult.VerifySpans` field and `finishRun`, which collects the spans, adds a rework span per fixer and sets them through a deferred assignment;
  - in `verify.go`: `runVerify`'s spans parameter, `timedVerify` and `VerifyGateName`;
  - in `shedadapters/weaver.go`: `weaverVerifySpans` and the `GateSpans` it sets on the done and stuck pointers, plus the `doc.go` sentence.
  - Renaming the inner `result` variable to `stopCause` in `finishRun` is forced by C4, because the new named return `result` would otherwise be shadowed.
  - The `weaver_test.go` subtests that were renamed or added, and the span assertions in `run_test.go`, are forced test support.
- **C5, tests for the unreached paths** → G4.
  - Third rung: the new `FailedAttemptsClimbToTheThirdRungWhichRecordsTheBatchDone`.
  - Lost strand: `MachineryLossRetriesOnceOnTheSameRungThenClimbs` became the `LostStrandRetriesOnceOnTheSameRungThenClimbs_*` table, with the new `paneDies` row and its fixture flag.
  - Resume: the new rows `FailedSecondRungAttemptRunsTheThirdRung` and `LostLastRungAttemptRetriesOnTheLastRung`.
  - Failed plan-level verify: span assertions added to the existing fixer-clears test and the exhausted-ladder test.
- **Commit `897b3639b`** (a doc-comment rewrap) only re-lines comments the range already changes, so it is part of C1 to C4 and adds nothing new.

**Per decision:**
- **G1:** built by C1 and C2.
- **G2:** built by C3.
- **G3:** built by C4.
- **G4:** built by C5. The C3 and C4 tests are the coverage of their own changes, not G4's paths.

**Findings:** none. Every change traces to a goal point or is forced by one, and every goal point has a change that builds it.

Outside this lens: G3 asks for spans "as the Webster rows do", and Webster's entries can carry `held` spans while Weaver's never do. Whether that matters is a question of what the spans mean, for the handler or the correctness lens, not a scope gap.

Tool calls: none.
