```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1 and F2 as the fixer carried them out.

**F2 (fixed as proposed): no findings.** The new `stopBlocks` comment, "Stop claims that handed out mail in its current turn", matches `stopBlockCap`'s comment. It also matches the code, because `ClaimAtBoundary` returns before counting when only an echo is handed out.

**F1 (own change plus the seam): one finding.**

- **LOW:** `server_test.go` row `AMarkedHookWaitWhoseClientLeavesAfterTheRingWasQueuedLeavesTheMailUnread` does not deterministically fail when `awaitOutcome`'s `Cancel` wiring is broken.
  - The hook ends with `_ = conn.Close()` and returns straight into `awaitOutcome`'s `select`. At that point `waiter.C` is certainly ready, but `watcher.closed` is ready only if the watcher goroutine has already seen the EOF.
  - Case 1: the watcher is late. The `select` takes the `waiter.C` ring branch. The write to the dead peer fails, `awaitAck`'s read fails, and `reading.Release()` runs. The row passes even if `defer waiter.Cancel()`'s drain is deleted.
  - Case 2: the watcher wins the race and both cases are ready. `select` picks at random, so the `closed` branch (the drain path under review) runs about half the time.
  - So the row detects a missing drain only intermittently. The review asked for the queued outcome to be ordered before the close, and the seam orders only the queue, not the watcher's notice.
  - The row's `Fatal` assertions, `read --wait` returning the message, are sound. The weakness is that the drain branch is hit by chance.
  - Evidence: the hook body in the new row, and `server.go` `awaitOutcome` lines ~395-427 (`case outcome := <-waiter.C` against `case <-watcher.closed`).
  - A deterministic variant would have the hook wait until `watcher.closed` is closed, but the hook receives only `outcomeQueued`.

**Proposal part (b), not added:** it closes the client between the ring write and the ack, which the existing row `AMarkedHookWaitWhoseClientLeavesBeforeTheAckLeavesTheMailUnread` already covers. Adding it again would be a redundant test, so skipping it is sound.

**Daemon-stop variant, not added:** the review's (a) also names "or the daemon stops". The `d.done` branch gets no row. It was not covered, and the report should say so.

**Tool calls:** none. The context already held every changed file and the rubric.
