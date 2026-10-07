---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-shuttle-agent-activity.md
    summary: The new RunState PID, written by Runner.start and reconstructAndWait, is pinned by no test, and its run and attach test files are not among the card's targets, although the whole liveness reading depends on it.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-run-shed-waits.md
    summary: The card says arm tests change only where the call now waits, but names only the awaiting and done tests; the existing running, halted and blocked-state tests that pin one Sleep and a returned Stuck from an unchanged child are not named, and the awaiting-decision case is listed as new although the named resume tests already cover it.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/07-state-change-notices.md
    summary: The notice tests name four existing tests but not TestRenderNotice, the table that pins the line format the new since, history and report fields change, nor the other existing notice tests the new Notify signature and episode key touch.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/01-orch-compact-boundary.md
    summary: The row for a boundary with no turn end after it duplicates what TestCompact_CompletesOnBoundaryAfterEntryAndIdle already pins, and the at-entry case is kept as its own renamed test instead of an assertion beside the before-entry case that test already holds.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/03-driver-notify-rule.md
    summary: The bound names the notices of cards 7 and 8 as what a restarted batten sends for every stop not yet announced, but only card 7 announces stops; card 8 sends the quiet and api-error notices of a running child.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/05-run-shed-waits.md
    summary: internal/battencli/arm.go still says the running watch appends one entry per poll, bounded by its 1440-bounce budget, both of which this card removes, and arm.go is not among the card's targets.
---

### [MEDIUM] RunState PID has no covering test

**Location:** `_lyx/plan/04-shuttle-agent-activity.md`, Intent paragraph on `RunState.PID`, and the Edit list.
**Issue:**
The card adds `RunState.PID` and has `Runner.start` and `reconstructAndWait` write it.
A run counts as live only with a live pid, so a start that fails to record it makes `ReadAgentActivity` return nothing and silently disables both card 8 notices.
The planned tests build `run.json` fixtures by hand with an injected `isAlive`, so none of them reaches the write.
`run.go`, `attach.go` and `rundir.go` are edited, but `run_test.go`, `attach_test.go` and `rundir_test.go` are not among the targets.
The plan template says a card carries the coverage for the behavior it introduces.
**Fix:**
Add `internal/shuttleengine/run_test.go` and `internal/shuttleengine/attach_test.go` to the Edit list.
State that `TestRunner_Start_HappyPath` gains an assertion that the persisted `run.json` carries `os.Getpid()`, and that the existing attach test that reconstructs a running record (`TestAttach_StartedSeededTrue`, or the test that already reads the persisted record back) gains an assertion that the pid is rewritten.
Neither adds a test function.

### [MEDIUM] Shed-waits card understates which existing tests change

**Location:** `_lyx/plan/05-run-shed-waits.md`, the "Tests" paragraph.
**Issue:**
The card says the arm behaviors keep their existing tests, "which change only where the call now waits instead of returning", and then names five awaiting and done tests and the fold test.
The statuses fake in `innerrun_test.go` repeats its last status forever and `Sleep` is a fake clock, so every existing test that ends on a `Stuck` returned from an unchanging child would now wait without returning.
Those tests are not named: the `StillRunningIsStuck`, `AbsentStatusSpawnsThenRunningIsStuck`, `BlockedIsAWait`, `PausedIsAWait` and `FailedIsAWait` rows of `TestInnerRun_VerdictTable`, `TestInnerRun_HaltedStatesWaitExemptWithoutSpawning` (which asserts one Sleep and an exempt `Stuck`), `TestInnerRun_AwaitingWithoutApprovalWaitsExempt`, `TestInnerRun_RunningArmReviewWaitNote`, and the halted-episode tests in `innerrun_halted_test.go` that count Warns and revives per call.
Without that list the writer of the code will add the new cases beside stale tests instead of rewriting them, which `PATTERN-test-economy` forbids.
The new-case bullet "an awaiting child's decision record is acted on within one check, and the call returns once the child leaves `awaiting`" is behavior `TestInnerRun_AwaitingResumesOncePerApproval` and its siblings already pin, which the card itself names as rewritten for the decision resume.
**Fix:**
Name the existing tests and rows above as rewritten to the in-call-wait premise, saying what each now asserts (for example that the unchanged-state rows become the change-driven returns the new cases describe, and the halted tests drive a state change or a pause to make the call return).
Fold the awaiting-decision bullet into the named awaiting tests as an assertion and drop it from the new-case list.

### [LOW] Notice tests do not name the table that pins the line format

**Location:** `_lyx/plan/07-state-change-notices.md`, the "Tests" paragraph.
**Issue:**
The card adds `since`, `history` and `report` to every notice line, but `TestRenderNotice` (the table that pins the rendered line, including the prefix and one-line rule) is not named.
`TestNotice_ReturnToRunningStartsNewEpisode`, `TestNotice_DeadDriverWhileRunning` and `TestNotice_NilNotifyRunsNoStep` also change with the `Notify` signature and the `condition|state|since` key, and are likewise unnamed.
The card's new-case list therefore cannot be told apart from rows that belong in those tests.
**Fix:**
State that `TestRenderNotice` gains rows for `since`, `history`, `report <path>` and `report none yet`, and that the three other tests change only for the new signature and key.

### [LOW] Compact-boundary card lists a duplicate row and a separate at-entry test

**Location:** `_lyx/plan/01-orch-compact-boundary.md`, the "Tests" bullets.
**Issue:**
`TestCompact_CompletesOnBoundaryAfterEntryAndIdle` already lands a boundary with no turn end after it and asserts completion, so the row "a boundary with no turn end after it" adds nothing; only the row for a boundary followed by an assistant turn end is new.
The same test already pins that a boundary before the phase entry does not complete, so the at-entry case is one more boundary value of that behavior, not a test of its own.
**Fix:**
Keep one new row, the boundary followed by an assistant turn end, and say that the no-turn-end case is the existing body of the test.
Make the at-entry case an assertion beside the before-entry one in that test and delete `TestCompact_BoundaryAtEntryCompletes` instead of renaming it.

### [LOW] Driver-notify card cites the wrong sibling cards for stop notices

**Location:** `_lyx/plan/03-driver-notify-rule.md`, the paragraph "What this lets through".
**Issue:**
The paragraph bounds the silence by "the batten notices of cards 7 and 8, which a restarted batten sends for every stop not yet announced".
Stop announcements are card 7's state-changed notices; card 8 sends the quiet and api-error notices of a running child, which announce no stop.
**Fix:**
Cite card 7 alone.

### [LOW] Stale comment about the per-poll watch in arm.go

**Location:** `_lyx/plan/05-run-shed-waits.md`, Edit list; `internal/battencli/arm.go`, the comment on `maxStatusHistoryEntries`.
**Issue:**
The comment says "The running watch appends one entry per poll, bounded by its 1440-bounce budget".
Card 5 removes both the per-poll entry and the budget, and updates the parallel comments in `commitstatus.go` and `docs/overview.md`, but `arm.go` is not a target, so the comment would go stale.
**Fix:**
Add `internal/battencli/arm.go` to the Edit list and say its comment is rewritten for the in-call wait, where history grows only on a state change, a pause or a failed status stat.

## Focus notes

- Silence cards (focus 1): card 3's "What this lets through" paragraph and card 6's Bound both state the batten that dies after the render and the orch strand that disappears after it, with their bounds (the pid-liveness test at render, the unwatched-then-adopted duplicate, a restarted batten sending unannounced notices), and neither points at a section of the decision record. Not a defect, apart from F5's sibling citation.
- Recipe blindness (focus 2): card 3 adds both notify stencils to the scan in `cmd/lyx/driverstencil_test.go` over the same token list, and the scan is line-based, so it also covers each stencil's header comment; the card's wording names no recipe, `lyx batten` or `lyx loom`, and the watched stencil says nothing of who announces a stop. Not a defect.
- Sibling references (focus 3): every mention checked against the Card Index; the only mismatch is F5.
- Planned tests (focus 4): F2, F3 and F4; card 8 names `TestNotice_QuietOnlyForRunningChildWithLiveDriver` as rewritten and lists only new behavior, so it needs no change.
