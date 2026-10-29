```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I paired the record's decisions and constraints and checked each pair, including its failure and restart paths.

**Pairs checked**
- D1 × D2 (both are `BoundaryBlock` output, from one renderer; D1 drops `systemMessage`, D2 keeps it): held.
- D1 × D3 (Stop count): held. A ring resets creel's count to 0, so the woken turn's first Stop sees a known count below `stopBlockCap`. This holds even though the 2.1.312 wake is enqueued with `stopHookActive:!0`, which leaves the count un-reset at that Stop.
- D1 × D3 (the merged Stop command and the waiter run in parallel at each Stop): held. A D1-continued Stop records no turn end, so the worker reads a turn in progress and does not ring a waiter registered mid-turn.
- D2 × D3 (reservations from a tool-boundary claim and from a ring claim): held. `unreadLocked` excludes reserved ids, so neither claim takes the other's mail.
- D3 × D4 (rewake text = `rewakeMessage` + space + stderr lines): held. The wake's first line opens with `[creel]`.
- D3 × D4 bound (an old session or a long-lived old `hook-wait` meeting a new daemon): held. An unmarked waiter gets no mail and writes the bell line as today; old settings keep the default label until relaunch, which the bound states.
- D3 × D5 (`acknowledged`/`released` against the ring's end paths): held. A lost confirmation logs `acknowledged`, and every pre-journal end logs `released`.
- D3 × D5 (an empty ring claim): held. It does not ring and logs at `Debug` as "nothing claimable".
- D1 × D5 (a Stop ack failing after the write): held as stated. It logs `released` while the model saw the mail, which the Constraints exception covers.
- D3 × Constraint "8 KiB budget fits every field": held. 8192 bytes plus the prefix is under 10,000 UTF-16 units, and the record says rewake output is not truncated.
- D3 × Constraint `PATTERN-tolerant-records`: held. An older daemon ignores the mark and rings with no lines, so `hook-wait` exits 0 and the bell follows.
- D3 × the existing bell window (claim reserved until ack): held. Reserved ids drop out of `hookWakeState`, so no bell fires during the reservation, and a release brings back the same set, unanswered, under the 30 s window.
- D1/`hook-read` × D3 against the Constraint "each message delivered once": **broken** (inconsistent policy; finding 1).
- The D1 to D5 acceptance criteria against their decisions: held. The D3 "exactly one operator copy" criterion is consistent with the Open risk that returns an undrawn body to discussion.

**Findings**

1. LOW, design. The two hook paths resolve an acknowledgement failure after the write in opposite directions.
   - **Where:** `decision-record.md` D3, second paragraph ("a failed write or a failed acknowledgement exits 0") and its second Bound, against the Constraints bullet on "Each message is still delivered once".
   - **The two choices:**
     - `hook-read` (D1/D2) writes first, so a failed ack means a duplicate: the mail was shown and stays unread.
     - D3's `hook-wait` decides after the ack and exits 0 on any ack failure. When the daemon had already journaled the read, the mail is lost as read.
   - **What D3 does not weigh:** exiting 2 on an unconfirmed ack would trade that loss for at most a duplicate, matching the `hook-read` exception. D3 never compares it, and the support log's Rejected alternatives cover resuming the wait, not exiting 2.
   - **Why it matters:** the Constraint lists the two exceptions side by side with no reason why the ring alone accepts silent loss over a visible duplicate.
   - **Evidence:** `internal/creelengine/client.go:253-262` (the ack is written, then `awaitAckConfirmation` returns `connLostError` on a lost confirmation); `internal/creelengine/server.go:470-474` (the daemon journals via `reading.Ack()`, then writes the confirmation).

Outside my lens, in one sentence: today's `Client.CallWatching` reconnects and resends a `Wait` request after a `connLostError`, including one from `awaitAckConfirmation` (`internal/creelengine/client.go:144-155`, `176`), which runs against D3's "never resumes"; that belongs to the existing-paths lens.

Tool calls: none; my context already held everything this lens needed.
