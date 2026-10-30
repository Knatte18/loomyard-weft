---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:414-446
    summary: wake gates and pushes the task head before it reads the pull request status, so a pull request merged at an ungated head is recorded as gated and ends Done instead of halting
    fix: In wake, read the pull request status first (pollStatus) and skip gateHead when the state is merged or closed, or compare the merged head against the gated head the record held before this wake's gate, so judge reaches its merged-with-other-head halt (prreview.go:616-617) as D9 requires. Add a row to TestPRReview_Wait for a pull request merged at an ungated task head that would pass the gate.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:144-173
    summary: the no path removes the pull-request record before Call runs endSession, so a session left live by an interrupted earlier call is never stopped
    fix: In closeRecordedPullRequest, call p.stopRecordedSession() before pullrequest.RemoveRecord and log a failed stop as endSession does, so Done stops the session as D7 says.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prsession.go:80-94
    summary: when StartSession succeeds and the record write fails, the new session stays live and unrecorded, and the re-entry starts a second one
    fix: In ensureSession, when updateRecord fails after a successful StartSession, stop result.StrandGUID through pr.Sessions.StopStrand (logging a failed stop) before returning the error.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/pairteardown/pullrequest.go:75-77
    summary: the gone-worktree Warn does not name the task branch as D12 requires, and fires for runs that never opened a pull request
    fix: Add the task branch name as a log attribute and word the message so it does not claim a pull request exists.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:315-327
    summary: the holdWhileCovered doc says an unreadable pause record is logged once, but the Warn fires on every read
    fix: Log the Warn once per hold (a local flag), or change the doc comment to say it is logged on every read.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/landingshed/prreview_wait_test.go:240-245
    summary: no row pins an open pull request with the continue flag set on a clean tree at a head other than the gated head, which must keep waiting without Close
    fix: Add a row to TestPRReview_Wait with state open, Continue set, clean tree and task head different from the record's gated head, asserting no Close call and no verdict.
---

### [MEDIUM] Merged-at-ungated-head halt is bypassed by gating first

**Location:** internal/landingshed/prreview.go:414-446
**Issue:** `wake` runs `gateHead` before the status poll. When the pull request was merged on GitHub at head M that the row never gated (an agent commit carried by the detached push) and the task head equals M, the gate passes, `GatedHead` becomes M, and `judge` sees merged head equal to gated head and returns Done. D9's state table and acceptance criterion require a halt naming both heads for a merge at a head other than the record's gated head. The row also gates and pushes onto an already merged or closed pull request.

### [LOW] No-path leaves an earlier session running

**Location:** internal/landingshed/prreview.go:144-173
**Issue:** The record is removed before `endSession` looks the session up through it, so `stopRecordedSession` finds nothing. A session live from an interrupted earlier call, then a mode that turns no, is orphaned although D7 says Done stops it.

### [LOW] Session started but unrecorded on a failed record write

**Location:** internal/landingshed/prsession.go:80-94
**Issue:** After `StartSession` succeeds, a failing `updateRecord` returns an error with the session live and unrecorded; the re-entry finds no strand and starts another.

### [LOW] Gone-worktree Warn lacks the task branch

**Location:** internal/pairteardown/pullrequest.go:75-77
**Issue:** D12 says the Warn names the task branch; it carries only the slug and a generic phrase, and fires for runs that never had a pull request.

### [LOW] Doc comment contradicts the pause Warn

**Location:** internal/landingshed/prreview.go:315-327
**Issue:** The doc says a `PauseCovered` error is logged once; the code logs it on every read.

### [LOW] Open, continue, clean, ungated branch unpinned

**Location:** internal/landingshed/prreview_wait_test.go:240-245
**Issue:** The `!gated` half of the guard at prreview.go:641 has no row; dropping it would pass the existing open-state rows.

## Rejected

- Lens code-correctness F4 (a mailed head is never re-gated): D8 asks for one mail per failing head and says nothing of re-gating after an environmental failure; the proposed backoff is beyond the answer key.
- Lens code-correctness F5 (session started for a closed, not-merged record): the code comment states it is intended for the agent's clean-up under a continue flag, and card 13 carries it.
- Lens code-correctness F8 (stop and resume around a continue poll): the continue flag is the operator's ending and the plan has the row stop the session before acting on it; not a defect.
- Lens code-tests finding 2 (no row for a failing RemoveRecord or ReadRecord on the ClosedForContinue re-entry): an internal error path with no trust boundary, which the testing rules do not require a row for.
- Lens code-affected-set: no findings. Declared PATTERN changes land with their declared text, no `Pattern-Departure:` trailers are present, and no named-account clause remains (`pr_account` appears nowhere outside the run's own records).
