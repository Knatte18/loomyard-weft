# Fixer-report, code review round 1

## Changes

Every finding but F6 is fixed as proposed.
F6 is `decision` class: the parent's decision #477 (keep the exemption, add the sentence) was recorded, and the sentence is in `internal/landingshed/doc.go`.
Commits: `7f54cd5e` (parentlock), `755e7eb4` (landingshed), and a third for the self-check fixes.
Findings share files, so commits group by package, not one per finding.

- F1: `judgeFixerChange` routes the `Baseline == head` branch through `letThrough`, so `Judged` is recorded. Row added: fixer commits nothing, the next merge brings parent commits, no `LineChange` call.
- F2: rows added for a no-op merge and for a merge whose `TaskHead` differs from `Judged`; both keep the old Baseline (`countingResolver` gained `onMerge`).
- F3 and F8 (approved route): a test over a passing gate reads the lock record during the fix review (absent), and asserts `Reviewed`, `Judged` and the moved `Baseline`.
- F8 (accepted route): a test keeps the acceptance's `Judged` across a refused push and asserts the second Call measures from the merged head.
- F4: `Waiter.pause` reports a closed changes channel and `Acquire` drops it. Tests: `pause` unit test, and an `Acquire` test that pins the drop through the context each sleep uses.
- F5: `Call` starts at the front when the attempt record's `Judged` is set (`returningFromFixer`); the bound (the record keeps it until a Done) is in `doc.go`. A test queues the run behind an older ticket.
- F7: test for the fixer inside the merge-in-required retry (requeue, lock freed during the fixer, lock held at each landing pass).
- F9: `TestTryAcquire_FrontTicketsAreGrantedOldestFirstAheadOfOrdinaryOnes`.
- F10: `Waiter.Watch` seam; a test where it fails asserts the poll grants and the Warn logs once.
- F11: `Store.Covered` comment says "first live ticket".

## Disputed

- Self-check fork finding (fix-consistency, MEDIUM), `PATTERN-daemon-wakeups.md` does not state the closed-watch fallback. Ground: premise false. Line 13 of the pattern states the parent lock wait polls from one second doubling to thirty seconds with an event wake on top; a dropped watch leaves exactly that poll.

## Deferred, with reason

- Self-check fork findings (fix-consistency MEDIUM, fix-affected-set LOW): the same missing `ok` check on the `<-changes` receive in `internal/landingshed/prreview.go`, `internal/loomshed/samplersession.go` and `internal/shedverbs/loopwait.go`. The review scoped F4 to `internal/parentlock/acquire.go`, and these files lie outside this round's write surface. Whether their channels can close was not checked; the parent decides on a separate change.

## Self-check

- fix-code-correctness, MEDIUM: the `Acquire` closed-watch test passed with the defect in place. Fixed with my own change: the test now records whether each sleep ran on the caller's context, which fails if the closed channel is not dropped.
- fix-code-correctness, LOW: garbled `judgeFixerChange` doc sentence. Fixed (also reported by fix-consistency LOW and fix-proposal LOW 1).
- fix-consistency, MEDIUM: twin `<-changes` sites in other packages. Deferred, above.
- fix-consistency, MEDIUM: pattern file. Disputed, above.
- fix-proposal, LOW 2: `Judged != ""` also holds after a halt that kept the lock. Fixed as the reviewer allowed: the bound is stated in `doc.go`.
- fix-affected-set, LOW 1: `PRReview.pause` twin. Deferred, above.
- fix-affected-set, LOW 2: no finding on the F6 sentence or `doc.go` lines. No action.

All four forks returned reports.

## Gate repair

The gate refused the round before any test ran: the `Waiter.pause` doc comment broke a sentence across two lines.
It now holds one sentence per line (`internal/parentlock/acquire.go`); no other file changed in this repair.
Gate: `lym gate test ./internal/landingshed ./internal/parentlock` passed (709 targets, 0 failed).
