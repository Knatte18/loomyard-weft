---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/compare.go (RunCompare), internal/driftengine/spawn.go (Active)
    summary: RunCompare takes the compare marker with one non-blocking take, so a reader probe holding it at that instant makes the compare refuse with a false already-running error
    fix: In RunCompare take the compare marker with the bounded-retry acquire the capture uses (lock.AcquireWriteLockWithin with a few seconds of retry, as RunCapture does with CaptureRequest.LockRetry), add a LockRetry field to CompareRequest that production fills with the same bound, and refuse with the already-running error only when the marker is still held after the retry. Add a TestRunCompare row where a probe holds the marker briefly while the compare starts and the compare still runs.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/driftengine/records.go (Acceptance.Ruling, the Ruling const group, RulingUnruled, RulingRuled)
    summary: The ruling doc comments say rulings apply under partial scope and to acceptances a caller has not ruled on, but the code sets them only under scope full for the agent session's acceptances
    fix: Reword the const group comment and the Ruling field comment to say scope full, and RulingUnruled to an acceptance the Drift-Verify agent session made that the run's parent has not yet ruled on, and RulingRuled to one the parent has ruled on or made itself.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-affected-set
    location: internal/pairteardown/cancel.go (the cancel path around the killLoop and endSession calls)
    summary: Only EndSession terminates the pair's drift capture job, so the cancel path never calls killCapture although the record says pairteardown ends the capture
    fix: Call t.killCapture(req.Slug) in the cancel path beside the killLoop call and return its error wrapped like EndSession does, with a Cancel row in teardown_test.go asserting the call; or, if cancel is judged to reach the capture through its runproc record, say so in the pairteardown doc comment.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/driftcli/cli.go (capture child body, the gitFacts read before RunCapture)
    summary: The capture record's Head is read before the capture lock is taken while Check re-reads HEAD later, so a HEAD that moves in between leaves a record naming a commit other than the captured code
    fix: Read HEAD after the lock is held, for example inside Check, and store that value in the capture record instead of the earlier read.
---

### [MEDIUM] Compare marker take has no retry window

**Location:** internal/driftengine/compare.go (RunCompare), internal/driftengine/spawn.go (Active)
**Issue:** D4 says the compare marker is probed as the capture lock is, and a reader's probe must never make a start refuse. RunCapture retries its take for a few seconds, but RunCompare takes the marker once. The loop's activity reading polls the marker through Active, so a probe can hold it at the instant the row's mechanical compare or an agent's lyx drift compare starts. That compare then fails with a false already-running refusal, which is an error stop of Drift-Verify.

### [LOW] Ruling doc comments name the wrong scope

**Location:** internal/driftengine/records.go
**Issue:** The comments describe rulings under partial scope and as a caller's, but Accept sets a ruling only under scope full, unruled for the agent session and ruled for a caller, matching D6 and D7. Also found independently by the handler and by the code-pattern-fit and code-affected-set lenses.

### [LOW] Cancel does not terminate the capture's named job

**Location:** internal/pairteardown/cancel.go
**Issue:** killCapture runs in EndSession only. The cancel path kills the loop and ends the session without it. The capture child is also ended through its runproc record, so a leak is unlikely, but the named job TerminateNamedJob reaches is left, against the record's constraint that pairteardown ends the capture.

### [LOW] Capture record Head read before the lock

**Location:** internal/driftcli/cli.go
**Issue:** The head stored in the record comes from a read taken before the lock and the precondition check. The window is narrow, but a moved HEAD would let the record name a commit the capture did not run on, and Drift-Await and Drift-Verify trust that field.

## Rejected

- code-pattern-fit finding on two readers of the branch point and dirty paths (driftcli versus loomcli): the lens could not show the readers disagree, and the record names SourceDirtyPaths only for the row seam; no defect evidenced.
- code-tests findings on missing Windows-only tests for the breakaway limit flag and CaptureJobName: the plan's shared decision accepts Windows behavior proved outside this hub's gates, and the DetachBreakaway flag test already exists.
- code-tests note on TestSkipList: the lens itself found it not redundant.
