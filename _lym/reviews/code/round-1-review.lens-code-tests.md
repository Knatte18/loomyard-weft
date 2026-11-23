```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that pins it.

**Behaviour → pinning test**
- A start resumes the newest earlier run when it died, carries a stop recorded as interrupted, or reads running with its waiter gone; it gets the preamble prompt, the predecessor's session ID and no skill-load turn → `TestStart_ResumesAnInterruptedRunsSession`, its three resumed rows.
- A start runs fresh after a run stopped on purpose, a timed-out run, a live waiter, a set-aside run, a run with no session, other output files, a refused resume check, or an engine without `SessionResumer` → `TestStart_ResumesAnInterruptedRunsSession`, its fresh rows.
- The earlier run is set aside once a start of the same output files is up, and is left alone for other output files → `TestStart_ResumesAnInterruptedRunsSession`, its `NoResume` assertion.
- A resumed start whose provider never came up sets aside both runs (`run.go:667-671`) → **none**.
- A resumed start leaves the carried instructions out of the prompt, still records them, and does not notify (`run.go:645-649`, `run.go:662`) → **none**. Every carry-over predecessor in `carryover_test.go` is a stop not marked interrupted, so every carry-over row runs fresh.
- A resumed run receives the resume check's warning and copied flag (`run.go:646`) → **none**.
- A step stop and an orphaned-step stop record the stop as interrupted → `TestRunner_StopStepRuns_StopsOnlyTheStepsRuns` and `TestRunner_StopOrphanedStepRuns_StopsOnlyRunsWhoseWaiterIsGone`, their new assertions.
- `StopStrand` records a deliberate stop with `Interrupted` false (`stop.go:158`) → **none**. No `StopStrand` test reads `Interrupted`.
- A superseding attach records a running candidate's stop as interrupted (`attach.go:240`) → **none**. The mutation survivor confirms it.
- A stop that cannot be recorded removes nothing → `TestStopStrand_UnrecordableStopRemovesNothing`, which keeps its earlier behaviour.
- `RetireInterruptedRuns` marks only resumable runs and treats a missing root as zero → `TestRunner_RetireInterruptedRuns_SetsAsideOnlyResumableRuns`.
- `RetireInterruptedRuns` carries on past a failed write and joins the errors (`crashresume.go:136-139`) → **none**.
- The goto hook sets the runs aside once, and a failed set-aside does not fail the goto → `TestGotoAdmitted_SetsAsideInterruptedRuns`.
- The set-aside runs before the PR-Review reset, and the reset still runs after a failed set-aside → **none**. The test does not see the reset at all.
- `newestPredecessor` and `carriedInstructions` split → the existing `carryover_test.go` rows, which reach the changed functions.

**Findings**

1. MEDIUM · design · `internal/shuttleengine/run.go:667-671`
   Nothing tests that a resumed start whose provider never comes up sets aside both the failed run and the predecessor.
   If it broke, every later re-call would retry a dead session.
   A row in `TestStart_ResumesAnInterruptedRunsSession` should make startup die and assert `NoResume` on both run records.
   Evidence: `crashresume_test.go:290-301` has no failing-startup row, and `grep NoResume` finds no other test.

2. MEDIUM · design · `internal/shuttleengine/attach.go:240`
   Mutation survivor: nothing tests that a superseding attach records a running candidate's stop as `Interrupted` and a terminal candidate's not.
   Negating the comparison passes every test.
   An attach test asserting `record.Interrupted` on the removed running candidate would kill it.
   Evidence: the mutation report's `attach.go:240` survivor, and `grep Interrupted` in the shuttleengine tests hits only `stop_test.go:343,468`.

3. LOW · design · `internal/shuttleengine/run.go:645-649` and `run.go:662`
   Nothing tests the resumed start's handling of carried instructions: kept out of the prompt, still recorded in `run.json`, no notifier call.
   The carry-over test seeds only stops not marked interrupted (`carryover_test.go:81`), and the crash-resume test seeds no carried instructions.

4. LOW · design · `internal/shuttleengine/stop.go:158`
   Nothing tests that `StopStrand` writes `Interrupted` false, which is what keeps a deliberate stop from being resumed.
   `TestStop_RecordsTheStopBeforeRemovingTheStrand` and the other stop tests never read `Interrupted`.
   The predicate row "a run stopped on purpose starts fresh" seeds the record by hand, so it never runs this write.

5. LOW · consistency · `internal/loomcli/gotoresume_test.go:1`
   The file header says the test pins the set-aside running "before its PR-Review reset", but the test never observes the reset or the order.
   It also does not show that the reset still runs after a failed set-aside.
   Either a reset seam should record the order, or the header should claim only what is pinned.
   Evidence: `gotoresume_test.go:1` and lines 158-167.

6. LOW · design · `internal/shuttleengine/crashresume.go:136-139`
   Nothing tests that `RetireInterruptedRuns` carries on past a failed write and returns the joined errors.
   The test covers only the all-success scan (`crashresume_test.go:383-386`).

No finding for the `crashresume.go:92` survivor: the mutated line only decides whether a warning is logged.
No finding for the `run.go:651` survivor either: an awaited spec never has the empty output set an unawaited predecessor declares, so deleting the `!spec.Unawaited` clause changes nothing observable.

Tool calls:
1. Searched the shuttleengine and loomcli tests for `Interrupted`, `NoResume`, the preamble, `CarriedInstructions` and the goto hook. My context held only the diff's own tests, not the existing ones that might assert these.
