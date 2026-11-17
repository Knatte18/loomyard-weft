---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: "internal/weaverengine/run.go:510"
    summary: "A failed strand stop records the stop as kind batch whatever the attempt's outcome, so a NEEDS_PARENT attempt re-enters through the batch branch."
    fix: "In runBatch (run.go:510-512), when stopErr is not nil, derive the stop kind from the attempt's outcome the way nextAttemptRung does: record stopKindNeedsParent with the attempt's rung for a needs_parent outcome, and for any other outcome record kind batch at the rung nextAttemptRung would have chosen (not the failed attempt's own rung unless it is the last). Add a TestRun row where StopStrand fails on a NEEDS_PARENT attempt and the re-entry starts on the rung that asked, never taking the no-spawn path."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: "internal/weaverengine/reset.go:128"
    summary: "Reset runs refuseForeignPaths before the strand probe, and own holds only paths failed attempts recorded, so a live attempt's uncommitted edits refuse the reset with the wrong advice and --stop-strand is unreachable."
    fix: "In Reset, call settleRecordedStrands before refuseForeignPaths. When settleRecordedStrands stopped at least one strand, add every path DirtyTracked returns to own, since the stopped attempt's edits are the run's own. Without --stop-strand a found run still refuses first with the strand's way forward. Add TestReset rows for a live attempt with uncommitted edits, refused without --stop-strand and reset with it."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: "internal/weaverengine/run_test.go"
    summary: "No test runs a rename through Weaver, so nothing fails if the rename rewrite of pending cards or the re-partition after it breaks (D3, D11 acceptance)."
    fix: "Add a TestRun row where the first batch's report carries a rename that a pending card names, and assert the second batch is partitioned and prompted from the rewritten card and that done cards are matched by slug."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: "internal/weaverengine/run_test.go"
    summary: "No multi-batch run test asserts that only the run's true last batch is built as last, which the later-reference check depends on (D3, D11 acceptance)."
    fix: "Extend the two-batch TestRun row, or add one, so the first batch's recorded gate spec and prompt carry LastBatch false and the final batch's carry LastBatch true, with a removed symbol a pending card names refused during the first batch."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: "internal/weaverengine/run.go:556"
    summary: "No test fails the state write after an attempt's outcome is recorded, so D6's rule that the strand is stopped all the same and the row returns Stuck naming the fault has no row."
    fix: "Add a TestRun row whose state save fails after the attempt's outcome is recorded, asserting the strand was stopped, the result is Stuck naming the fault, and no next attempt started."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: "internal/weaverengine/run.go:687"
    summary: "stopReason truncates the cause at a fixed byte count, which can cut a StrandStopError's closing lym reed remove way forward that D6 requires the stop to name."
    fix: "In stopReason, keep the truncation from touching a cause that is a StrandStopError: truncate only the leading cause text and append the way forward after it, or truncate on a rune boundary before the way forward. Add a stuck-reason assertion with a long stop error."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: "internal/weaverengine/reset.go:96"
    summary: "Reset probes the run lock and releases it at once, so a row stepped before the archive runs against a half-reset tree."
    fix: "Hold the run lock for the whole Reset: acquire it with lock.TryAcquireWriteLock in place of RunActive, refuse when not acquired with the same way forward, and release it by defer."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: "internal/weaverengine/resume.go:323"
    summary: "An attempt whose outcome is fault climbs a model rung on the next step although the fault says nothing about the model."
    fix: "In judgeOpenBatch, treat a last attempt with outcome fault like a lost attempt that spent no allowance, so its next attempt runs on the same rung, and add a TestRun_Resume row for it."
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: "internal/weaverengine/run.go:272"
    summary: "The beginBatch doc comment and the State.Stop field doc both say a batch begin clears the stop record, but the code never does and re-entry relies on that."
    fix: "Edit the beginBatch doc comment (run.go:267-273) and the Stop field comment in state.go:40-41 to say the begin leaves the stop record alone, which the re-entry's Reentered marking needs, and that only a batch recorded done and a passing verify clear it."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: "pattern/PATTERN-cwd-resolution.md:38"
    summary: "The bound-packages list names websterengine and burlerengine but not weaverengine, the new engine that takes absolute paths."
    fix: "Add weaverengine to the Bound packages list in pattern/PATTERN-cwd-resolution.md beside websterengine."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: "internal/weaverengine/reentry_test.go:41"
    summary: "No row pins that re-entry refreshes the stopped batch's own card hashes so an edit to its card text carries into the last-rung attempt."
    fix: "Add a TestRun_Reentry row that edits the stopped batch's own card before re-entry and asserts the last-rung attempt's prompt carries the edited text and the record's CardHashes hold the new hash."
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-tests
    location: "internal/weavercli/wiring.go:117"
    summary: "The reset verb's dependency wiring (state history, plan read at a records commit, code reset, dirty tracked paths) has no test, only the envelope and flag are covered."
    fix: "Add a tier-appropriate test in internal/weavercli that builds resetDepsFor over a hermetic git fixture and drives Reset through ResetCode, ReadPlanAt, StateHistory and DirtyTracked."
---

### [MEDIUM] Failed strand stop turns a NEEDS_PARENT stop into a batch stop

**Location:** internal/weaverengine/run.go:510
**Issue:** `runBatch` records `stopKindBatch` for every failed strand stop. D7 says a NEEDS_PARENT stop resumes on the rung that asked and never takes the no-spawn path, and the resume path in `resume.go` keeps that. The run path and the resume path disagree.

### [MEDIUM] Reset's foreign-path check precedes the strand probe

**Location:** internal/weaverengine/reset.go:128
**Issue:** A live attempt's uncommitted edits are not in `own`, so the reset refuses with advice that is wrong while an agent still writes, and `--stop-strand` cannot be reached. D2's refusal order names the probe, not a tracked-path check, ahead of any move.

### [MEDIUM] Rename through Weaver has no test

**Location:** internal/weaverengine/run_test.go
**Issue:** The acceptance criterion for renames rewriting pending cards has no failing row. A grep for rename in the weaverengine and runplan tests finds none.

### [MEDIUM] True-last-batch rendering has no test

**Location:** internal/weaverengine/run_test.go
**Issue:** `IsLastBatch` is reached in weaverengine tests only by `render_test.go` setting `LastBatch` directly, so a mapped partition without the pending cards would not fail any row.

### [MEDIUM] State-write failure after a recorded outcome has no test

**Location:** internal/weaverengine/run.go:556
**Issue:** The acceptance criterion says the strand is stopped also when the state write fails. Only failed-stop and failed-commit rows exist.

### [LOW] Stop reason truncation can cut the reed remove way forward

**Location:** internal/weaverengine/run.go:687
**Issue:** The 600-byte cut falls at the end of the cause, where `StrandStopError` puts `lym reed remove <guid>`.

### [LOW] Reset does not hold the run lock

**Location:** internal/weaverengine/reset.go:96
**Issue:** The probe releases the lock before the archive, leaving a window for a stepped row.

### [LOW] A fault outcome spends a rung

**Location:** internal/weaverengine/resume.go:323
**Issue:** `nextAttemptRung` treats `fault` like a failure and climbs, although the fault is Weaver's own.

### [LOW] Stale doc comments say a batch begin clears the stop record

**Location:** internal/weaverengine/run.go:272
**Issue:** The code leaves the record alone, and `runBatch` relies on that to mark `Reentered`.

### [LOW] PATTERN-cwd-resolution omits weaverengine

**Location:** pattern/PATTERN-cwd-resolution.md:38
**Issue:** Every other engine this diff's siblings touch is listed. No test pins the list, so the doc is the only place the omission shows.

### [LOW] Re-entry's refresh of the stopped batch's own card hashes is unpinned

**Location:** internal/weaverengine/reentry_test.go:41
**Issue:** Only a pending-card edit has a row.

### [LOW] Reset's CLI dependency wiring is untested

**Location:** internal/weavercli/wiring.go:117
**Issue:** `TestResetVerb` covers the envelope and flag, not the seams the verb hands the engine.

## Rejected

- Lens code-correctness finding 6, `Run` takes no context: no decision, card or acceptance criterion asks for mid-run cancellation, and the lens did not check how the driver ends a producer.
- Lens code-correctness finding 8, an upper-case full `head_sha` fails the first-parent walk: `git rev-parse HEAD`, which the report contract names, prints lower case.
- Lens code-affected-set finding 2, the tapestry recipe still says Webster's auto-rebaseline: the acceptance criterion asks only that the Plan-Review text name the shared path, and it does.
- Redundancy candidates `TestCancellation_RealProducersReturnErrorNotStuck`, `TestGateRows_StopsNameAWayForward` and `TestWebsterProducer_ConflictRerouteCapThenRepeatHalts`: the diff only updates their constructor signatures or conflict-directory paths and adds no test, so there is no new test to fold.
- Redundancy candidates `TestImplementerEntry_ForwardsTheRestOfTheRowConfig` and `TestImplementerEntry_WebsterBuildsTheWebsterProducer`: the Weaver rows pin that `implementerEntry` forwards the row config to `weaverEntry` for the weaver setting, which `TestWeaverEntry` and `TestEveryEntry_RejectsUnknownConfigKey` reach only by calling the constructors directly.
- A declared-PATTERN-changes check: the directive declares none, and the range carries no `Pattern-Departure:` trailer.
