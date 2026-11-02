```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
1. **A release ends the poll delay after the floor.** Pinned by `TestAcquire_ReleaseEndsThePollDelayAfterTheFloor` (`internal/parentlock/acquire_test.go`). It uses the real fswatch and the delays `[1s, 1s]`. The branch where the floor has already passed (`rest <= 0`) has no row, because the injected `Now` is constant.
2. **Watch fails to open, so the wait polls only, with a Warn.** No test pins it. `fswatch.Watch` is not injectable, so no row reaches the `logger.Warn` branch or `changes == nil` in `pause`.
3. **Watch channel closed mid-wait, and the poll ceiling still bounding the wait.** No test pins it. A closed `changes` channel makes `pause` return after the floor every time, so the delay never backs off. No row exercises it.
4. **Front ticket queued ahead of an ordinary ticket.** Pinned by `TestAcquire_FrontWaiterIsGrantedAheadOfAnOlderTicket` and the `parentguard_test.go` row "a wait at the front is granted ahead of a ticket queued before it".
5. **Ordering of two front tickets, and a front waiter waiting behind an older front ticket.** No test pins it. These are the `liveTickets` sort tie-break and the `tryAcquire` rule "waits only behind an older front ticket". No row has two front tickets, and `List`'s `Waiters` order is unasserted.
6. **Front ticket cleaned up on every exit.** Covered only through `TestAcquire_EndsWaitWithTicketRemoved`, which uses a non-front waiter. The cleanup goes through the same `abandonTicket` path.
7. **`parentGuard.ensure` passes `Front` through.** Pinned by the new `parentguard_test.go` row. Parent-Align's `false` is covered by the existing rows.
8. **A failing verify frees the lock for the fixer, and the run merges and verifies under the lock again.** Pinned by the new `TestFinalize_GateAndFixer` subtests in `finalize_gate_test.go`. They assert the lock is absent during the fixer's turn, the merge count, and that the lock is gone after the landing.
9. **A second failed verify spends only the attempts left.** Pinned by the second row, with `wantAttempts` `{2,1}`. Attempts running out across passes, ending in `endSpent` after a requeue, has no row.
10. **Rebase of the Baseline after a merge that brings parent commits.** Pinned by both rows, through `wantLineChanges` (`t2..f1` and `t2..f1,f1..f2`). Three branches of `rebaseAfterParentMerge` have no row:
    - a no-op merge (`merge.UpToDate`) leaving the Baseline in place;
    - `!found`;
    - a `TaskHead != Judged` mismatch.
11. **`judgeFixerChange` treats Baseline == HEAD as no change.** Covered only implicitly, by row 1 making no `LineChange` call after the rebase.
12. **`Judged` recorded by `letThrough`.** Only the within-caps route is exercised. The fix-review-approved and operator-accepted routes have no row asserting `Judged`, nor that a later merge moves the Baseline past them.
13. **A fix review frees the parent lock.** No test pins it. No test asserts the lock is free while the review session runs, though the change, the `doc.go` text and the summary all claim it.
14. **A Finalize with no lock store frees nothing and stays in one pass.** Covered implicitly by the existing struct-literal tests, which use the zero guard. No row names it.
15. **The `lockFreed` requeue after the merge-in-required retry (`finalize.go:327`).** No test pins it. No row has a fixer run inside the retry's `mergeAndGate`.
16. **Re-check tests moved to the parent-records commit.** The release-before-landing cases and the retry case (`at == 2`) are still pinned. The old "release during the fixer's turn" scenario can no longer occur, since the fixer runs outside the lock. Nothing is lost beyond that.

**Findings**
- **MEDIUM, design:** `internal/landingshed/finalize.go:844` (`freeLock` before `runFixReview`). The fix-review lock free is untested. A regression putting the review back under the lock would pass every test. Add a row in `finalize_gate_test.go` that runs an over-cap fixer change and reads the lock record during the review session.
- **MEDIUM, design:** `internal/landingshed/finalize.go:725` (`rebaseAfterParentMerge`). The no-op merge branch (`UpToDate`) is untested, and so is the case where the merge's `TaskHead` differs from `Judged`. The summary says the verify is skipped over the same tree when the merge is a no-op. Add rows with the resolver answering `AlreadyUpToDate`. They should assert that the Baseline and `Judged` are unchanged and that `LineChange` is measured from the old Baseline.
- **LOW, design:** `internal/landingshed/finalize.go:824` (`letThrough`). `Judged` is only asserted indirectly, and only on the within-caps route. Add an assertion on the approved route (`Reviewed == head`), and one on the accepted route.
- **LOW, design:** `internal/parentlock/store.go:240` and `:283`. Two front tickets are ordered oldest first, and a front waiter waits behind an older front ticket. Neither has a test. Add a `TestAcquire` row with two front waiters.
- **LOW, design:** `internal/parentlock/acquire.go:115`. The watch-open failure Warn and the poll-only fallback have no test. Making `watch` injectable on `Waiter` or `Store` would allow one.
- **LOW, design:** `internal/parentlock/acquire.go:76`. A watcher channel that closes mid-wait collapses the backoff to repeated one-second wakes. It is untested and probably unintended.
- **LOW, design:** `internal/landingshed/finalize.go:327`. The requeue inside the merge-in-required retry has no row.

**Tool calls:** none. The manifest already held every file this lens needed.
