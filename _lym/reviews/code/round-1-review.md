---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/tendercli/runintents.go:31
    origin: lens:code-correctness
    summary: resumeAside sets Cancel aside, which also drops a finished cancel, so a resume of a cancelled run reads running and acts instead of waiting on terminal
    fix: In internal/runfsm/act.go Judge, make Aside.Cancel drop only an unfinished cancel intent (clear r.Batten.Cancel only when its Finish is nil), so a finished cancelled record still derives Cancelled and the terminal exemption in tenderengine Resume applies; add a TestAct Judge row for Aside.Cancel over a finished cancel.
  - id: F2
    severity: LOW
    class: design
    location: internal/battencli/cancelmarker.go:53
    origin: lens:code-correctness
    summary: An undecodable batten record reads as an unfinished cancel, so lym batten run names a cancel as the way forward that fails on the same decode error
    fix: In cancelMarkerRefusal (internal/battencli/cancelmarker.go), when the reading's Intent is empty because the record did not decode, word the refusal to name the unreadable record and its repair by hand, not a re-run of lym batten cancel; add a row to TestCancelMarker for the undecodable record asserting that wording.
  - id: F3
    severity: LOW
    class: design
    location: internal/tendercli/cli_test.go
    origin: lens:code-tests
    summary: The refusal of lym tender resume for a run another machine owns is pinned for pause only, and TestResume uses a fake verdict
    fix: In TestPauseAndResume_WriteLiftAndRefuseOverTheHub (internal/tendercli/cli_test.go), add a resume of the slug owned by another machine that asserts the refusal names that machine and the lym adopt way forward, driven through the real ReadRun and Judge.
  - id: F4
    severity: LOW
    class: consistency
    location: internal/battencli/cancelmarker_test.go:152
    origin: handler
    summary: Three tests the redundancy report lists as contained pin what their covering tests cannot see and carry no testtiming keep marker
    fix: "Add a //testtiming:keep line directly above each func Test line: TestCancelMarker_ReadsNothingFromAPrimeWithNoScratchRoot in internal/battencli/cancelmarker_test.go, reason: pins that a read of a prime with no scratch root creates no lock file or scratch directory; TestJudgeCancelMarker in the same file, reason: pins the race rows of the three probes, the intent finished between the read and the shared lock and a lock held before the intent is written; TestLog_ChangesOnly in internal/dobbyengine/passes_test.go, reason: pins the changes-only log lines of the pass loop, which the covering tests never assert."
---

### [LOW] Resume of a finished cancel reads running

**Location:** internal/tendercli/runintents.go:31, internal/runfsm/act.go:186
**Issue:** `Aside.Cancel` sets `r.Batten.Cancel = nil` whatever the intent's finish. A finished cancelled record then falls through `Derive`'s `Cancelled` and `Cancelling` conditions and reads `running` or `starting`, so `lym tender resume` of a cancelled run answers `act` and the terminal exemption in `Resume` is dead for it. D3 has the resume set aside the run's own cancel intent when it asks, which is a cancel under way.

### [LOW] Undecodable record names a cancel that cannot run

**Location:** internal/battencli/cancelmarker.go:53
**Issue:** `readUnfinishedCancelIntent` reads an undecodable record as unfinished, so a torn record refuses `lym batten run` with "run lym batten cancel to finish it". That cancel reads the same record through `ReadStatus` and fails on the decode error, so the named way forward cannot work.

### [LOW] Resume's owner refusal is untested

**Location:** internal/tendercli/cli_test.go
**Issue:** D3 requires a resume of a run another machine owns to be refused naming that machine. Only `pause` has a test over the real reader; the `TestResume` rows use a fake verdict, so a regression of the hand-over answer on resume fails no test.

### [LOW] Contained tests carry no keep marker

**Location:** internal/battencli/cancelmarker_test.go:152
**Issue:** The redundancy report lists `TestCancelMarker_ReadsNothingFromAPrimeWithNoScratchRoot`, `TestJudgeCancelMarker` and `TestLog_ChangesOnly` as contained. I compared their assertions with the covering tests.
`TestCancelMarker` never stats the lock file or scratch root, and its rows do not drive the finished-between-reads and lock-before-intent races that `TestJudgeCancelMarker` pins.
The covering dobby tests do not assert the pass loop's changes-only log lines. Each is accepted and needs the keep marker.

## Rejected

- code-tests finding 2, no test drives the `lym fabric remove` entry: the lens could not confirm it within its call budget, and `TestTeardown_CallSequence` and `TestRemoveCancelledRecord_DeletesOnlyACancelledRecordAndCommitsItsRemoval` pin the teardown path that `lym fabric remove` takes; a vague suspicion does not block.
- Cleanup's refusal for a cancelled record whose task branch stands only on origin names `lym batten cancel --remove` on the owner machine, not `lym fabric remove`: D3 names `lym fabric remove` for a pair that stands, and a pair standing only on another machine cannot be removed from here, so the wording is sound.
- The focus questions on `Derive`'s precedence, `Act`'s order, the cancel path's commit-before-end and finish state, `PATTERN-lym-files` text landing, `PATTERN-leaf-packages` entry and the origin-owed marker staying: read against the code and settled as correct; no finding.
