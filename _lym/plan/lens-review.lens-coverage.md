```yaml
lens: coverage
outcome: reported
```

Coverage lens: each decision, criterion and branch of the plan against the planned test rows.

**Findings**

1. **MEDIUM — D3, "the ring's claim marks the ring answered at the acknowledgement" (card 4, row `AMarkedWaiterIsRungWithTheBudgetedClaimAndItsAckJournalsItAndAnswersTheRing`).** This row can't fail when Ack does not answer the ring. It sets a running turn before probing for the bell. A probe that reads a turn in progress already marks the ring answered: `internal/creelengine#worker.wakeHookStrand`, the branch `unanswered && !idleness.NoTurn` → `store.markRingAnswered`. Fix: model the row on `ARingAnsweredByAClaimIsNotFollowedByTheBell` in `internal/creelengine_test#TestScheduler_HookWake`. Use an idle reading whose turn end is before the ring, leave #2 claimable, and register a second waiter after the Ack. Then assert no bell over 45 s. With no live waiter, the 5 s grace bell fires for the set {#2} whether or not the ring was answered.

2. **MEDIUM — D6, the ring-Ack echo keeps the lines of ids "skipped as no longer the holder's unread mail" (card 7).** No row reaches the skipped branch. `AJournalFailureAtTheRingsAckEchoesOnlyTheIdsBeforeTheFailingWrite` covers only journaled ids. In `internal/creelengine#Reading.Ack`, the `msg == nil || msg.Holder != r.holder || msg.Status != StatusUnread` → `continue` branch is never driven for a ring reading. Fix: add a row where a ring-claimed message is withdrawn or forwarded before the Ack, and assert that its line is still in the echo.

3. **MEDIUM — D3, "Every way the ring's answer path ends before the daemon journals the acknowledgement releases the claim" (card 4).** The new branch in `daemon.awaitOutcome` that releases the reading when the ring response's write fails has no row. The release-on-cancel row (`ARingOutcomeNeverTakenIsReleasedWhenItsWaitIsCancelled`) drives `Waiter.Cancel` at the store level only. It never reaches `awaitOutcome`'s `watcher.closed` or `d.done` arms with an outcome waiting, which is the race the record's plan-writer note names. The Shared Decision `release-on-cancel` argues the daemon-level path holds through `defer waiter.Cancel()` rather than testing it.

4. **MEDIUM — D5, "each bell send logs … the send's outcome" (card 9).** `TestStore_ClaimLog` plans rows for `sent` and `not-landed` only. The `busy` and `failed` outcomes that card 9's Intent adds to `internal/creelengine#worker.typeBell` have no row.

5. **MEDIUM — D5, the ring path's nothing-handed-out Debug line (card 9).** Card 9's Intent has `ringHook`'s empty or paused ring claim log the Debug line with path `ring`. No `TestStore_ClaimLog` row reaches it; its ring rows are "acknowledged" and "released" only. The ring's `ack-failed` line is also unreached, and D5 says it carries no echo ids.

6. **MEDIUM — D5, the `ack-failed` line's `dropped_echo_ids` attribute (card 9).** The claim ack-failed row (`FailJournalWrites(0)`) is not stated to hand out an echo, so no row asserts `dropped_echo_ids`. D5 requires the line to carry "the ids of the echo it handed out, which D6 drops".

7. **LOW — D3, the pause-read-error branch of `ringHook` (card 4).** "With a pause or a pause read error it rings nothing and leaves the waiter registered." Only the paused row is planned; no row sets `SetPausedError` at the ring.

8. **LOW — D3, a marked ring starts the strand's Stop count over (card 4: "Either ring starts the strand's Stop count over").** Only the unmarked rows `ARingStartsTheStopCountOverWhereTheDaemonStartCapRefusedMail` and `ARingAfterTheStopCapStartsAFreshCount` reach it; no row uses a marked waiter.

9. **LOW — D3, "What the claim leaves past the budget stays unread with its flag standing" (card 4).** The budget row checks that a later claim hands out #2, but no row asserts the flag still stands after the ring's Ack. Card 7's flag row covers the echo only.

10. **LOW — D3, "a ring whose claim comes back empty does not ring" (card 4).** The empty-claim branch is unreachable apart from the pause. `claimLocked` always takes the first of `claimableLocked`, and the locked step has already checked that set equals `ids`, which is non-empty (`wakeHookStrand` returns early on `len(ids) == 0`). The criterion is met only through the pause row; the branch as written has no reaching input.

11. **LOW — scope (card 7).** "A pause no longer returns before the locked step" also moves the Stop-count reset (`boundary.Stop && !boundary.StopHookActive` → `stopBlocks = 0`) under a pause. The decision record does not license that change, and no row pins either behaviour.

Tool calls:
- Read `internal/testkit/logcapture/logcapture.go`, to check whether card 9's Debug-level capture is available. My context held only its function list.
