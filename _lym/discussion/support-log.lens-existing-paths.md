```yaml
lens: existing-paths
outcome: reported
```

I traced the paths D1–D5 meet in the code that runs today. Two of them break D3 and the record says nothing about either.

**BLOCKING — D3: an old waiter acknowledges the ring's mail without ever showing it.**
- A `hook-wait` process lives for up to a day (`creelWaitTimeout`), so after `lym` is updated an old waiter can still be running.
- When the binary is replaced the daemon exits, and the old client resends the same `hook-wait` request to the new daemon (`Client.resume`: "A read and a hook-wait resume by sending the same request again").
- When the new daemon rings it with reserved mail and `AckRequired`, the old client's `exchange` acknowledges any such response on its own when `handle` is nil (client.go:253–265).
- The old `hookWait` then writes `creelengine.BellLine` instead of the mail and exits 2. The mail is journaled read, but the session only gets the bell. Its later Stop has nothing to hand out, so that mail is never delivered.
- This happens to every idle hook session on every deploy. It breaks the record's constraint "Each message is still delivered once".
- Fix: the hook-wait request carries a mark meaning "this waiter takes mail". The daemon claims for a ring only when the mark is there, and rings an unmarked waiter as today, with no mail.

**MEDIUM — D3: the client's reconnect can deliver the ring's mail twice.**
- `Client.Call` resumes a `hook-wait` on any lost connection, and that includes a failed write of the acknowledgement (`conn.Write(ack)` → `connLostError` → resume).
- Under D3 the handle has already written the lines to stderr. The daemon releases the reservation and the client registers a new waiter.
- The next ring then writes more lines to the same stderr and exits 2. The model sees the first lines in that wake, and sees them again when a later claim hands out the released mail.
- The record says only that "a connection that drops before the acknowledgement releases the reservation".
- Fix: say that once a ring carrying mail has arrived, `hook-wait` never resumes. If the acknowledgement fails it exits 0, so Claude Code drops the stderr and doesn't wake. The mail stays unread and the bell's window for an unanswered ring applies.

**MEDIUM — D3: an old daemon's ring reaches a new client with no mail.**
- During a deploy the new `hook-wait` binary can get a ring with no lines from a daemon still on the old binary.
- The record doesn't say what the client does then: exit 2 with empty stderr (an empty wake) or exit 0 (no wake, and the bell follows). The window is about a second, but the rule should be in the record. Recommend exit 0.

**LOW — D3: a code comment the record's doc list misses.** The comment on `OutcomeRing` in `internal/creelengine/pool.go` says a ring "carries no mail and claims nothing", which D3 makes false. The record's doc list names only the three `doc.go` files.

**Paths traced, per decision:**
- **D1**
  - Merged Stop command (exit 0 after output records no turn end): survives.
  - Events-file turn-end readers (idle reading, `TurnEndedAt`): survive.
  - Stop count with `stop_hook_active` and the cap of 6 against Claude Code's 8: survives.
  - Smoke test's one-Stop-payload check: survives.
  - The claudeengine doc lines 134–136 describe the old rendering, but the record already lists that doc for update.
- **D2**
  - Tool-boundary claim resetting the Stop count: survives.
  - PostToolUse activity stamp: untouched.
- **D3**
  - `wakeHookStrand`'s ring/bell decision after a released ring: survives. No turn end comes after the mark, so no new ring fires, and the bell fires once the 30-second window passes.
  - Marking the ring answered by the ack or by a busy probe: survives.
  - `ringHook`'s Stop-count reset: survives.
  - Same-set mark after a release: survives (the bell covers it).
  - Server `awaitOutcome` needs a hand-out step; that is plan work.
  - Client resume and the old-client auto-ack: break (the findings above).
- **D4**
  - Settings are rendered at launch, so a session launched before the change keeps the old waiter entry and shows the default labels. D4's bound covers that.
- **D5**
  - The claim verb and the ring both run in the daemon, where its logger lives: survives.
  - A hook that never reaches the daemon goes unlogged; D5's bound already says so.

**Tool calls:**
1. Grepped Go code for readers of hook output text, `BellLine` and `OutcomeRing`, to find paths that read the changed state; my context had only the files I had opened.
2. Grepped `deliver.go` and `verbs.go` for the ring-answer and no-turn logic, which my context had only from the docs.
3. Grepped `client.go` and `server.go` for reconnect, ack and hook-wait handling, which my context lacked.
4. Read the client resume code, `wakeHookStrand` and the server's hook-wait handler, to trace the release and resume paths.
5. Read the client's ack branch and the server's `awaitOutcome`, to confirm that an old client acknowledges by itself when `handle` is nil.
