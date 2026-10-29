```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the creel-mail-delivered-once plan, covering the failure and recovery ends each card adds.

Per refusal or recovery: the next step it names, and whether it progresses.

- Card 3: the Ack journal write fails, and the ids it left unjournaled are released. The next step is a later claim or read handing them out. Progresses.
- Card 4: the ring's pause read finds a pause or an error, so it rings nothing and keeps the waiter. The next step is the next probe after the unpause. Progresses.
- Card 4: the ring outcome's write fails, or the ack never comes, so the reading is released. The next step is the bell after the 30 s window. Progresses.
- Card 4: an outcome is left in the waiter's channel at the client's close or the daemon's stop, and `Waiter.Cancel` releases it. The next step is the bell, or a resumed waiter on the new daemon. Progresses.
- Card 5: the ack is unsent, so `ErrAckUnsent` is returned and hook-wait exits 0. The next step is the daemon's EOF release, then the bell. Progresses.
- Card 6: the write of the lines fails, so no ack is sent and the exit is 0. The next step is the daemon's release, then the bell or the next claim. Progresses, but its test row races (finding 1).
- Card 7: an echo claim is released and the echo stays owed. The next step is the next claim, run because the flag stands; Release syncs the flag. Progresses.
- Card 8: an echo-only Stop exits 1. The next step is the merged command recording the turn end. Progresses.

Findings:

1. MEDIUM — Card 6, subtest `hook-wait whose stderr refuses the lines exits 0 and leaves the mail unread`.
   - The step it asserts next (the following tool-boundary `hook-read` prints the mail) races the daemon's release.
   - The ring's reading is released only when the daemon's `awaitAck` `readLine` sees EOF from the closed connection, in a daemon goroutine (card 4's `daemon.awaitAck`; today's `internal/creelengine#daemon.handOut` releases the same way).
   - `hookWait` returns as soon as the client closes, so a `hook-read` run at once can find #1 still reserved: the claim comes back empty, and the subtest flakes.
   - The row needs a wait that ends on the release, such as a `read --wait` as a strand or polling the journal or claim, before asserting the mail.

2. LOW — Card 4, `TestDaemon_Protocol` row `AMarkedHookWaitWhoseClientLeavesBeforeTheAckLeavesTheMailUnread`.
   - "a strand read hands the message out" races the same asynchronous release unless the read is `Wait: true`.
   - The existing `ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread` row uses `read` with `Wait: true` for exactly this reason (`internal/creelengine_test#TestDaemon_Protocol`).
   - The card should say so.

3. MEDIUM — Card 4, the "empty claim rings nothing and keeps the waiter registered" branch.
   - This is D3's criterion "a ring whose claim would come back empty does not ring".
   - No input reaches the branch, and no row names one:
     - `worker.wakeHookStrand` returns early on empty ids (`internal/creelengine#worker.wakeHookStrand`).
     - `ringHook` proceeds only when the current claimable set equals the non-empty ids.
     - `claimLocked` always takes the first message, so the locked claim is never empty.
   - Either name a reaching input and row, for example an echo or hold state that empties the set between the check and the claim, or write the branch as a fail-loud wiring guard and say so in the Intent.

4. LOW — Card 4, row `AMarkedWaiterIsRungWithTheBudgetedClaimAndItsAckJournalsItAndAnswersTheRing`.
   - Its "with a running turn set, 45 s of probes type no bell" passes even if `Reading.Ack` never answers the ring.
   - A probe that reads a running turn after a ring calls `markRingAnswered` itself (`internal/creelengine#worker.wakeHookStrand`, the `unanswered && !idleness.NoTurn` branch).
   - The assertion does not show that the ack answers the ring. The probes after the ack need a no-turn reading with a turn end recorded after the ring, so that only the ack can have answered it.

5. LOW — Card 8, `hookRead`'s write-failure error on an echo-only claim.
   - On an echo-only claim it still reads "could not write the mail, so it stays unread" (`internal/creelcli#hookRead`, `writeBlock`).
   - The echo is what failed to show, and per card 7 it stays owed, not unread mail.
   - The card does not reword the message for the echo case, so its stderr line misstates what happened and what comes next.

No further tool calls were made.
