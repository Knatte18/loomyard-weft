---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/orchengine/watcher.go:1500
    summary: A failed InjectOutcome re-read after a successful Withdraw discards the withdrawn list, so a withdrawn resume line never has its pause-record mark cleared
    fix: In withdrawQueuedCompact, in the error branch of the post-withdraw InjectOutcome call (the one inside `if st.Pending.ID != 0`), set st.ResumeLineWithdrawn = st.ResumeLineWithdrawn || slices.ContainsFunc(withdrawn, func(message CreelWithdrawn) bool { return message.Subject == pauserecord.ResumeLine }) and call w.save(*st) explicitly before returning w.holdOnCreelFailure(st, err), because creelFailed skips its save when State.Stuck already holds the same text. The next call then clears the mark through clearWithdrawnResumeLine or the retried clear block. Update the doc comment's "a creel failure leaves the phase as it was" claim if the saved field makes it untrue.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/orchengine/watcher_compact_test.go:232
    summary: The new post-withdraw InjectOutcome failure branch has no test
    fix: Add a row or subtest to TestCompact_BoundaryWithdrawsAStillQueuedCompact that scripts a withdrawn resume line in e.c.withdrawn, makes the second InjectOutcome call fail (failNext("injectoutcome", err) arranged so the pre-withdraw read succeeds), and asserts the phase stays compacting with the step pending and the resume-noticed mark cleared by the next tick, which fails until F1 is fixed.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: scope
    location: internal/orchengine/watcher_compact_test.go:244
    summary: No row ends the wait on CreelUnconfirmed, though the summary and doc.go name unconfirmed as an outcome that completes the phase
    fix: Add the row {"Unconfirmed_CompletesWithNoFurtherWithdraw", CreelUnconfirmed, 0, 1} to the table in ACompactInADeliveryAttemptHoldsTheBoundaryUntilItSettles.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: scope
    location: internal/orchengine/watcher_compact_test.go:232
    summary: No test combines a withdrawn resume line with an in-flight compact, the path that runs ClearResumeNoticed and then answers errCompactInFlight
    fix: Add a row to the in-flight subtest that sets e.c.withdrawn to a resume line with a noticed resumed entry and asserts the mark is cleared after the first, held tick while the step stays pending.
    origin: lens:code-tests
---

### [MEDIUM] Failed re-read after a successful withdraw loses the withdrawn resume line

**Location:** internal/orchengine/watcher.go:1500
**Issue:** Withdraw has already removed a queued resume line from creel when the second InjectOutcome call fails.
The function returns through holdOnCreelFailure before the withdrawn list reaches ClearResumeNoticed, and nothing records the line in State.ResumeLineWithdrawn.
The next tick's Withdraw finds nothing, so the pause record's ResumeNoticed mark stays set and the orch's resume line is never queued again.
No call sat between Withdraw and the clear before this diff.

### [MEDIUM] No test for the post-withdraw re-read failure

**Location:** internal/orchengine/watcher_compact_test.go:232
**Issue:** The existing hold tests fail `withdraw` or `sent`, none fails `injectoutcome` after the withdraw, so the new branch and F1 are unpinned.

### [LOW] Unconfirmed outcome not covered

**Location:** internal/orchengine/watcher_compact_test.go:244
**Issue:** inFlight treats unconfirmed like injected by omission, and no row pins it.

### [LOW] Withdrawn resume line with an in-flight compact not covered

**Location:** internal/orchengine/watcher_compact_test.go:232
**Issue:** The existing resume-line subtests use a compact that completes in one tick, so the in-flight path through the clear block is unpinned.

## Rejected

- The Debug line logging on every in-flight tick without an Info at entry: PATTERN-log-levels permits Debug for a wait bounded by the handoff timeout.
- The fake's `Withdraw` comment naming creel's withdraw: it states the real behavior the fake mirrors, not a dependency on another symbol.
- The focus question on a pending `/compact` whose id creel never answered: it is withdrawn once and dropped, the boundary is consumed once, and the unchanged id-0 test pins it.
- The focus question on the handoff-timeout drop: the boundary branch returns before the generic timeout check, so the in-function timeout is the only bound, measured from the same `PhaseEnteredAt`; not a defect.
- The summary's Debug and Warn claims: the diff bears them out, one Debug per in-flight tick and one Warn at the drop.
