---
round: 1
focus:
  - "internal/landingshed/finalize.go, `Call` loop over `landUnderLock` and `lockFreed`: does every exit path of a pass that freed the lock either re-take it at the front and run again, or end Stuck; can a pass free the lock, fail to re-take it, and still reach a landing attempt without the lock?"
  - "internal/landingshed/finalize.go and parentguard.go: after the fixer, the board entry says the run merges the parent anew and verifies under the lock as on its first try. The summary says the verify is skipped over the tree the fixer already verified when the merge is a no-op. Settle which the code does, and whether a tree that differs from the one verified can land unverified."
  - "internal/landingshed/fixerattempts.go and finalize.go `judgeFixerChange`: the new `judged` head and the Baseline move on a merge. Does a Baseline equal to HEAD, or a merge of parent commits onto the judged head, ever hide a real fixer change from the caps, or count parent lines as the fixer's? Does a second failed verify spend only the attempts `finalize_fix_attempts` leaves, never a fresh cap, across every route into the fixer?"
  - "internal/parentlock/store.go and acquire.go, front tickets: ordering of front tickets against ordinary tickets, a stale or abandoned front ticket (crashed holder, released lock), and a returning run that never gets the lock. Is a front ticket cleaned up on every exit, including a cancelled wait?"
  - "internal/parentlock/acquire.go, the fswatch wake: a watch that fails to open or is closed mid-wait, a release between the last attempt and the watch start (lost wake-up), a leaked watcher on return, and whether the poll ceiling still bounds the wait when events stop."
  - "Lock re-check tests in finalize_recheck_test.go were moved from the fixer's turn to the parent-records commit. Does the moved release still exercise the re-check the test names, or has coverage of the release-during-fixer case gone?"
  - "Test economy: new tests in parentlock/acquire_test.go, parentguard_test.go and finalize_gate_test.go against the tests already covering the same behavior; any added behavior without coverage, such as the watch-open failure Warn, the fix-review lock free, or the attempts bound."
  - "Scope against the board entry: the diff also frees the lock for fix review sessions and moves the Baseline, neither of which the entry names. Judge each against the entry's design and `Left to the run`. Check that docs/overview.md, the package docs and pattern/PATTERN-daemon-wakeups.md say what the code now does, and that the commit messages and `_lym/landing/summary.md` claims hold against the diff."
---

Round 1 seed from the board entry in the decision record, the review-range commit messages and the change description.
The code under review is the lock handling in `Finalize` and the queue and wake changes in `parentlock`.
