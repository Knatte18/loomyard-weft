```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set, F1 and F2 of round 1.

**F2 (stale "Stop blocks" wording):** no findings. The search covers `internal`, `cmd` and `docs`, `.go` and `.md` files, and matches `stop block(s)` and `blocks of its current turn`. The only remaining hit is `internal/creelengine/verbs_test.go:1261`, a test message the review already rejected. The fixed `stopBlocks` field comment in `pool.go` was the only stale site.

**F1 (ring answer-path ends through `awaitOutcome`):** LOW. The fix covers only one of the three ends the finding names.
- Changed text: the new row `AMarkedHookWaitWhoseClientLeavesAfterTheRingWasQueuedLeavesTheMailUnread` in `server_test.go`, plus the `beforeOutcomeWait` seam.
- Covered: the client closing after the ring is queued, and (existing) the client closing between the ring write and the ack.
- Not covered: the finding's "or the daemon stops" end, the `<-d.done` branch of `awaitOutcome`'s select with a ring already queued. The finding's fix text asks for "before the connection closes or the daemon stops", and the new row exercises only the close.
- Not covered: a failed ring write through the real daemon. The new row only reaches it when the select happens to pick `waiter.C` after the close, so which branch runs is random and the failed write is not pinned.
- Evidence: the new row's hook calls only `conn.Close()`, and a search of the new row finds no daemon stop. The `server.go` select has both `case <-watcher.closed` and `case <-d.done`.
- Judgement call for the fixer: the `d.done` end can be reached with the same seam by stopping the daemon from the hook instead of closing the client. Release is then observable only after a restart on the same journal.

Tool calls:
1. One `grep` for `stop block(s)` and `blocks of its current turn` across `internal`, `cmd` and `docs`, to rerun F2's search for stale sites. My context had only the single edited site.
