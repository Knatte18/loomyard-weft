```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch and test row the cards add, I checked whether a real entry point can reach the state it guards and whether the guarded site recurs unguarded.

1. MEDIUM — Card 4, ring: the "empty claim rings nothing and keeps the waiter registered" branch is never reached, and the D3 criterion it carries has no planned row.
   - `worker.wakeHookStrand` returns before ringing when `len(ids) == 0` (internal/creelengine/deliver.go, the `wakeHookStrand` body).
   - `Store.ringHook` rings only while the claimable set still equals `ids` (internal/creelengine/verbs.go:439-444).
   - `claimLocked` always takes the first claimable message. So under the lock a non-empty, unchanged set can never claim nothing.
   - In practice an empty claim comes from the set changing (a concurrent claim reserved it) or from the pause read. Only the pause row covers either. No row reaches the set-changed early return with a marked waiter and asserts the waiter stays registered.
   - The card should either drop the empty-claim branch as dead code or call it a fail-loud wiring guard, and add a row for the changed-set case.

2. MEDIUM — Card 4, row `AMarkedWaiterIsRungWithTheBudgetedClaimAndItsAckJournalsItAndAnswersTheRing`: the "Ack answers the ring" branch is not reached by an assertion that fails without it.
   - The row sets a running turn, then checks that 45 s of probes type no bell.
   - A probe that reads a running turn marks the ring answered on its own: `unanswered && !idleness.NoTurn` calls `store.markRingAnswered` in `worker.wakeHookStrand`.
   - `worker.typeBell` also returns early when `!w.idle(strand)`.
   - So "no bell" holds whether or not `Reading.Ack` marks the ring answered. The check needs an idle strand (no running turn) and a remaining set, so that only the Ack's mark keeps the 30 s bell away.

3. MEDIUM — Card 9, D5 (`ack-failed` replaces the claim's Warn): the Warn the card narrows is no longer in the member the card edits.
   - Card 4 moves the acknowledgement half of `daemon.handOut` into the new `daemon.awaitAck`, including `Reading.Ack` and the `could not journal an acknowledged read` Warn (today internal/creelengine/server.go:470-472).
   - Card 9 says "`daemon.handOut` keeps its … Warn for a read alone" but lists only `internal/creelengine#daemon.handOut`, not `plan:internal/creelengine#daemon.awaitAck`.
   - Ring acknowledgements also go through `awaitAck`. Implemented as written, a claim's or a ring's journal failure logs twice: `awaitAck`'s Warn plus `ack-failed`.

4. MEDIUM — Card 6, new `TestCreelCLI` subtest `hook-wait whose stderr refuses the lines exits 0 and leaves the mail unread`: the released state may not exist yet when the next step runs.
   - After the handler's write error, `Client.exchange` returns and closes the connection (`defer conn.Close()`, internal/creelengine/client.go:347).
   - The daemon releases the ring's reading only when `awaitAck`'s read of the ack line returns EOF, on its own goroutine.
   - The following tool-boundary `hook-read` claims without waiting. It can run while the ring's reservation still stands and print nothing, so the row fails intermittently.
   - The fix is to wait for the release first, for example with a `read --wait`, or to poll the log.

5. LOW — Card 4, row `AMarkedHookWaitWhoseClientLeavesBeforeTheAckLeavesTheMailUnread`: the same ordering applies.
   - The release happens on the daemon goroutine after the client closes.
   - The row's "a strand read hands the message out" reaches the released state deterministically only if the read is `Wait: true`, as in `ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread` (server_test.go:338). The card does not say so.

6. LOW — Card 9, Debug line with path `ring` for "`ringHook`'s empty or paused ring claim": the empty half has the same reachability gap as finding 1.
   - The empty half is reached only through the set-changed early return.
   - `TestStore_ClaimLog` lists no ring-path `nothing` row, paused or otherwise, so no row reaches this logging branch.

Tool calls:
- Read `internal/testkit/creelfake/creelfake.go` lines 195-275, `Idle` and `Paused`: to confirm the `SetIdleHook` row in card 4 can set the pause between the worker's pause read and the ring's without a deadlock. My context lacked the fake's lock handling.
- Read `Store.noteStall` in deliver.go: to confirm a bell under `FailJournalWrites(0)` still sends. My context lacked whether the bell path journals.
