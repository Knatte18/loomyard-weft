---
round: 1
focus:
  - "`tickHandoff` in `orchengine`: does the timeout count only from the creel `read` of the request, on both the note path and the requested-cycle path, with unpaused time measured from the read? Settle whether any path still falls back to the post time or `CycleRequestedAt`."
  - "`State.RequestReadAt` lifecycle: is it cleared at every site where `State.RequestID` is cleared (cycle start, `enter`, `toIdle`, `ResetForFreshLaunch`), and is a restarted watcher's persisted value honoured without a new journal query?"
  - "`creelSeam.MailReadAt` in `orchcli`: does the journal fold return the first `read` entry for the exact id, ignore `reminded` entries and other mails' reads, and report zero when unread? Check the failure path goes through `creelFailed` and holds the timeout."
  - "Idle-phase stale-marker removal: does a pending request whose cycle never starts still get abandoned from its request time, as the board entry's intent and the summary claim?"
  - "Docs landing with the diff: do the `orchengine` and `orchcli` `doc.go` text and the `lyx orch refresh` and `lyx orch distill` help text match the new behavior, and does any PATTERN or other doc still describe the old post-time rule?"
  - "Tests: does each behavior the diff adds (`MailReadAt`, the read-time persistence, the unread-never-times-out case) have coverage, and does any new test duplicate an existing one's coverage?"
---

Seed focus from the board entry (timeout starts at the creel `read` event, not the post) and the change description's claims.
