---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/landnotice.go:flagMembers
    summary: A member whose run is past Land but whose entry is not yet done is re-flagged after its notice cleared the flag, so it can send a second landed line.
    fix: In flagMembers, skip any open member whose slug appears in the tick's listed runs with PastLand set, or whose slug has landing-notice memory; pass the runs (d.listedRuns) from landNotices into flagMembers for this. Add a TestLandNotices row with beam and pullRequest set where the entry stays open after the run passes Land and the notice has cleared the flag, expecting one line and no second flag call.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/beamcli/dobbydeps.go:238
    summary: The new PullRequestRecorded seam closure in hubDobbyDeps has no test, so the gate that decides flagging and the follow-up is unpinned.
    fix: In internal/beamcli/dobbydeps_integration_test.go, extend TestHubDobbyDeps_NotifyOnLandSeamsWriteTheHubBoardEntry (or add rows beside it) to call deps.PullRequestRecorded against a hub beam with no landing record, a record holding no pull request, and a record holding a pull request, asserting false, false and true with a nil error.
  - id: F3
    severity: LOW
    class: design
    origin: handler
    location: internal/dobbyengine/landnotice.go:flagMembers
    summary: A member that lands, and whose entry turns done, in the tick after the pull request is first recorded is never flagged, so its landing sends no line and no follow-up.
    fix: In landNotices, when followUp is not empty, count a run remembered ahead of Land whose entry is a member of this beam as requested in rememberRuns even when its entry is already done and unflagged, so its line goes out under per-task; keep the clear owed only when the entry still carries the field. Add a TestLandNotices row where the pull request becomes recorded on the same tick the run lands and its entry reads done, expecting the one landed line with the follow-up.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/landnotice_test.go:TestLandNotices
    summary: Rows are missing for flagMembers' filter and retry, the follow-up on the commit-unknown and done-entry lines, and land_notice off with a recorded pull request.
    fix: Add rows to TestLandNotices for a non-member and a done member that stay unflagged, a SetNotifyOnLand stub that fails once and then flags on the next tick, a landed-without-commit member line carrying the follow-up, a done entry taken up with beam and pullRequest set carrying the follow-up, and LandNoticeOff with beam and pullRequest set sending no line while the member is flagged.
---

### [MEDIUM] Second landed line when a flag is re-set after its notice cleared it

**Location:** internal/dobbyengine/landnotice.go:flagMembers
**Issue:** `PastLand` includes the teardown row, so a run can be past Land while its board entry is still open. The landed line is posted, `ClearNotifyOnLand` clears the flag and the memory is dropped. On the next tick `flagMembers` sees an open member without the flag and sets it again. When the entry later reads done, `takeDoneEntries` finds a flagged done entry with no memory and posts a second line. That breaks the goal that a landing sends one line. The window's width depends on when the entry turns done, which I did not verify.

### [MEDIUM] PullRequestRecorded wiring has no test

**Location:** internal/beamcli/dobbydeps.go:238
**Issue:** The seam's `found && record.PullRequest != nil` condition decides whether a beam's dobby flags members and adds the follow-up. The only test of the beam seams drives `SetNotifyOnLand` and `ClearNotifyOnLand`. A behavior the diff adds has no coverage.

### [LOW] Notice lost when the pull request is recorded in the tick the member lands

**Location:** internal/dobbyengine/landnotice.go:flagMembers
**Issue:** `flagMembers` flags only open members and `takeDoneEntries` takes only flagged done entries. A member whose run lands, and whose entry turns done, before the dobby first reads the recorded pull request is never flagged. Its landing goes into the beam with the pull request recorded and gets no line, where the removed announcement covered every landing.

### [LOW] Test rows missing around the new flagging

**Location:** internal/dobbyengine/landnotice_test.go:TestLandNotices
**Issue:** The member filter (other project, done status, kind), the failed-write retry, the follow-up on the commit-unknown line and on a done entry's line, and the `land_notice: off` decision recorded in the change description are each added behavior with no row that fails if it breaks.

## Rejected

- Flag outlives a dropped pull request (lens:code-correctness, LOW): the amended entry is silent on it, a dobby-set flag cannot be told from a hand-set one, and the extra line carries no wrong text.
- Flagging members with OwnedElsewhere (lens:code-correctness, LOW): the write is idempotent and harmless, and no rule in the amended entry or `pattern/` bars it.
- Scope against the amended entry: the dobby sets the flag on its next tick instead of `lym beam pr open` writing the board, which the change description names as a design decision; the one-trigger outcome and the follow-up on the flag's line are delivered, and the stale flag and announce paths are fully removed (code-affected-set found none left).
