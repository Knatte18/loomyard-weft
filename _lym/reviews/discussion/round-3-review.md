---
verdict: BLOCKING
prior:
  - id: F12
    status: resolved
    answer: D4 now has the row clear the fix-here count with the mark, parent_review_asked and the session id on a clean-tree reroute, the Bound says between two send-backs or before the first, and the criterion pairs a re-entered row with zero and a re-step with the kept count. The three texts agree, and the reset needs no segment.
findings:
  - id: F13
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:111
    summary: D4 clears the mark, fix-here count, parent_review_asked and session id inside the row's Call, before the shed persists the Reroute, so a crash in between loses the send-back and resets the count while D4 and its Bound say a crash-resume keeps the count and the cap holds
    fix: In D4, replace the sentence that has the row clear the mark, the fix-here count, parent_review_asked and the session id on a clean tree and then return Stuck with Reroute set, with a rule that the row on a clean tree returns Stuck with Reroute set and clears nothing. The clear happens at the start of the row's next Call, and only when the newest shed history entry of the PR-Review row is its reroute entry, which is the entry shed persists after the Call returns. A Call that meets the mark with any other newest entry (a re-step, a crash before the reroute persisted, the resume after the dirty-tree Stuck, the resume after a reroute-cap halt) keeps the count and the session id and reroutes again. In D4's Bound, replace the sentence that a send-back the reroute cap halts has cleared the count too, so each resume opens a fresh allowance, with one saying that the cap-halted row keeps the mark and the count and its resume reroutes again within the three reroutes the resume grants. Keep the other Bound sentences, and update the criterion about a re-entered row and a re-step so the re-entered row is the one whose newest history entry is the reroute. If the row cannot read the shed history, name the other way the row tells a re-entry after a reroute from a re-step, such as a send-back number the record carries and the reroute entry's record, and say how it is read.
---

### [BLOCKING] D4's clear runs before the reroute is persisted, so a crash drops the send-back and resets the cap

**Location:** `_lym/discussion/decision-record.md:111`, with the claims at lines 104-105 and 134-137.

**Issue:** D4 has the PR-Review row, on a clean tree, clear the send-back mark, the fix-here count, `parent_review_asked` and the session id, and then return a Stuck verdict with `Reroute` set (lines 110-112).
The clear is a write to the pull-request record inside the row's `Call`.
The reroute is persisted afterwards by shed in `rerouteStuck` (`internal/shedengine/reroute.go`), which appends the reroute history entry and moves `current_producer`.
The engine already treats a crash between a producer's work and that persist as a case: `internal/shedengine/run.go` states that `current_producer` stays put, so the next step re-calls the producer.
In that window the record has no mark, a zero count and no session id, while shed still has the run at PR-Review.
The resumed row finds no mark, so it does not reroute: the send-back the verb stored and recorded is lost, Code-Review never sees the findings, and the agent starts a new session with a fresh allowance of `pr_review_small_max` in-place fixes.
D4 says the opposite twice.
Line 105 lists a crash-resume among the cases that keep the count, and the Bound claims at most `pr_review_small_max` findings are fixed in place between two send-backs.
Both hold only for a crash before the clear.
The same ordering decides the reroute-cap halt: the clear has already run when `rerouteStuck` halts, so the resume the cap's way forward names finds no mark and does not reroute.
The Bound states that outcome for the count, but not that the send-back is dropped, and the cap's own way forward promises "three more reroutes" the resumed row cannot use for the send-back that was halted.
The focus question, whether a re-step, a crash-resume and the resume after the dirty-tree Stuck behave as the Bound and criteria state, is therefore settled for the re-step and the dirty-tree resume, which do keep the count, and is false for a crash after the clear.
Moving the clear to the row's next entry after the persisted reroute removes the window, keeps a reset event that needs no segment, and leaves the three D4 texts consistent.
