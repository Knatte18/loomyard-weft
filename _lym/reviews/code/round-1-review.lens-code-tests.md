```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests — for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour: pinning test.** I made one tool call: a grep of added test names and rows, which does not show test bodies, so every "none" below is "no visible test" and not confirmed absent.

- D1 tapestry rows and `plan_split` gate: `TestPlanSplitRows`, `TestSequence_PlanSplitRunsThePartsRows`.
- D1 settled-guard writer naming: five `settledguard_test.go` rows (latest Skipped, writer done after a skip, no history, both skipped, history unreadable).
- D2 config:
  - `tapestry_test.go`: the timeout-accessor and chair-timeout subtests are visible. The `parts_depth`, `parts_max` and model refusals are probably table rows I did not see.
  - `TestPartsPlanTable` pins the timeouts resolving against the effective plan timeout. The `plan_timeout_min == 0` case against `run_timeout_min` is not visible by name.
- D3 parts parser and check: `TestParse`, `TestCheck`, `TestValidate`.
- D3 `DecisionIDs`: `TestDecisionIDs`.
- D3 `NewPartsGate`: `TestNewPartsGate`.
- D3 `validate-parts` verb: `TestValidatePartsCmd`, plus the Darn refusal rows in `arm_seed_test.go`.
- D4 `Parts-Write` table: `TestPartsTable`.
- D4 `Parts-Write` rotator, commit and Stuck commit: `TestPartsRotators`, `TestPartsWriteEntry`.
- D5 writer kind in `Table.Validate`: `table_test.go` (+34 lines). The row names are not visible.
- D5 writer done line, failed end and held turn: `TestEngine_Run_WriterSeat`.
- D5 resume with writers: `TestEngine_Resume_Writers`. I saw the found-writer and lost-writer-passes-Accept subtests. A lost writer with outputs missing, or failing Accept, has no visible subtest.
- D5 hold deferral (`advisorWork.outstanding` counting a done writer mid-turn): none visible. `work.go` changed but the diff stat shows no `work_test.go`.
- D5 per-seat `Timeout` in `seatSpec`: `prompt_test.go` changed (+7 lines), content not visible.
- D6 `Parts-Plan-Write` table: `TestPartsPlanTable`.
- D6 writer-ended-step mapping to Stuck or error: `TestMultiLLMProducer_MapsAWriterThatEndedTheStep`.
- D6 `join` marking a writer `EndedStep` when the chair ends first: `TestEngine_Run_WriterEndDecidesTheJoin`.
- D6 rotators and `PartsPlanWriteEntry` commits: `TestPartsRotators`, `TestPartsPlanWriteEntry`.
- D6 `partsPlanCommitPathspec`: `wiring_commitstatus_test.go`, changed by +12 lines.
- D7 part scope:
  - `TestValidate_PartScope`, `TestValidateFormat_PartScope`, `TestValidatePart`
  - the `glyphchain_test.go` rows for `caller-uncovered`
  - the `Reparse` keeps-scope subtest
- D7 `validate-plan --part` exclusivity: four rows in `validate_test.go`.
- D7 `validate-plan --part` outside a `Parts-Plan-Write` recipe: probably `TestValidatePlanCmd_Part`, name not visible.
- D8 `planparser.JoinParts`: `TestJoinParts` in `join_test.go`.
- D8 `loomshed.JoinParts`: `TestJoinParts` in `partcheck_test.go`.
- D8 `join-parts` verb refusals: `TestJoinPartsVerb` has rows for a blocked run, another row, an absent status and a join refusal passed through. The row for a non-chair caller (including an empty caller) is not visible by name.
- D13 interrupt policy: two rows in `interruptpolicy_test.go`.
- D13 report timing phase: `timing_test.go` (+4 lines).
- D13 registry pins: `registry_test.go`.
- D12 stencil markers: `markers_test.go`. Exempt as static prose.

**Findings**

1. MEDIUM, class design. Location `internal/seatengine/work.go:46`. The hold deferral for a done writer (`run.ended() && !(Writer && finishedDone())`, plus the skipped up-line check for writers) has no visible direct test, and no `work_test.go` is in the diff. The D5 criterion "the chair's held turn sends the parent nothing while a done writer is mid-turn on a fix" needs a row in `engine_test.go` or a new `work_test.go`. Evidence: the diff stat lists no `work_test.go`, and the `engine_test.go` and `resume_test.go` subtests I saw do not name the hold deferral. Check this before keeping it.
2. LOW, class design. Location `internal/seatengine/resume.go` (lost-writer branch). No visible subtest for a writer not found whose outputs are missing or whose Accept fails (`ender.end(lost)`, result with `AcceptErr` set). Only the found-writer and Accept-passes cases are visible. Verify against `resume_test.go` in the diff.
3. LOW, class design. Location `internal/planparser/join.go:298`. `writeJoinedPlan` failure paths (removal or write error after old card files are deleted) have no visible test. `join_test.go` rows are all pre-write refusals. The "run it again" way forward is untested.
4. LOW, class design. Location `internal/loomcli/joinparts.go:234`. The non-chair caller refusal, including the empty-caller wording and the success envelope, is not visible by row name in `TestJoinPartsVerb`. Confirm a row exists; if not, it is uncovered.

Tool calls:
1. `git diff <range> -U0 -- '*_test.go'` filtered to added `Test` functions and row names, because my context held only the test file stat, not test contents.
