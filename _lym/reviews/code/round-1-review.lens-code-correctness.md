```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, run case by case over the diff. Those two spawned lines of the runner are my own turn; the report follows.

## Per-function list (case run → verdict)

- `inbound.poll`/`cursorFor`: no cursor, other-channel cursor, empty channel (clock timestamp), `Latest` failing with an API refusal. Correct. The cursor is rewritten before any read, and a refusal gets one notice per episode.
- `inbound.poll` loop: three messages where the 2nd is skipped and the 3rd fails. Correct. The skip moves the cursor, and the failure leaves it before the 3rd.
- `inbound.forward`: `Beams` error, unknown slug, 4097-byte body, `Post` refused, `Paused` error after an accepted post. Correct. `Beams` errors return before any post, the other three answer and move the cursor, the refused post goes to `retryOrGiveUp`, and the `Paused` error is logged once and counts as handled.
- `retryOrGiveUp`: a failure at t=0 and at t=5min. Correct. It is judged at the next poll after 5 minutes, and `failingSince` is lost across a relay restart, which only delays the give-up.
- `inbound.setPresence`/`status`: presence written but notice failing, presence not written. Correct.
- `SetPresence`/`WritePresence`: no file to `slack`, `slack` to `slack`, a corrupt value. Correct.
- `API.call`/`History`/`Latest`: 429, `ok:false ratelimited`, 5xx, `has_more` with an empty `next_cursor`, page 2 failing. Correct. Slack always sends a cursor with `has_more`, so the early-break case is unreachable.
- `compareTS`: a 10-digit against an 11-digit seconds value. Correct.
- `outbound.read`: nil ack, `errReadBudget`, `ErrWaitTimeout` with and without a handler run, rate limit, Slack failure. Correct.
- `outbound.post`, one 25-second budget with a 20-second request in flight: the handler can outlast creel's 30-second acknowledgement. Correct, because the ack is then refused or lost, the record keeps the handled ids, and the redelivered read posts nothing twice.
- `outbound.refused`: refusal counts 1 to 5, a transport error, the replacement line failing to post. Correct. The message is counted handled after the 5th refusal and transport errors count toward no cap.
- `outbound.post`, stale mail followed by a Slack failure on a fresh message. **Wrong, F1 below.**
- `outboundText`/`programSlug`: an orch beam sender, a three-segment program sender, EXTERNAL, another hub's program, a two-segment program. Correct.
- `RunRelay`/`takeRelayLock`: a re-exec with the same pid, a lock held by another relay, a first-read in-strand-pane refusal. Correct. The double release of the lock is a no-op.
- `firstRead`: refusal then success, in-strand-pane refusal, context ending while it waits. Correct.
- `runInbound`: re-exec failing with the lock not retaken, a rate limit, an error with activity. Correct. `EpisodeSet` is mutex-guarded (`internal/logger/episode.go:42`), so the two loops can share `relayLog`.
- `starter.ensure`: a first spawn, five young deaths in a row, a 3-day relay dying, `LastSlackOK` older than `LastSpawnAt`, a non-away start. Correct. The count reaches `OutcomeGaveUp` on the fifth failure, and the backoff shift stays at most 1&lt;&lt;4.
- `AwaitFirstRead`/`relayRunning`: a stale state from a dead relay, a live relay holding a refusal. Correct.
- `AwaitReparent`: a parent that cannot be read, the intermediate already gone. Correct.
- `spawnDetached`: a failed start, `wait` true and false. Correct.
- `slackcli` `login`, `setup`, `away`, `stopRelay`, `paneRefusal`/`callerPane`: an empty stdin, bot not a member, a rerun after a failed stop, an orch pane, an unknown pane. Correct.
- Token and setup files: `fsx.AtomicWriteBytes` uses `os.CreateTemp` (`internal/fsx/fsx.go:60`), which creates the file mode 0600, so the owner-only claim holds on Unix.
- `fabriccli.moveAnchor`/`fail`/`restartRelay`: failure after `EndDobbies`, record standing, record unreadable, prime against beam, success. Correct. The relay restart now runs on every path after `EndDobbies` and is joined into the error.
- `beamcli.ensureSlackRelay`, `orchcli.startRelayStep`/`relayStarter`: gave-up, a start error, a beam, nil. Correct.
- `orchengine.WithPresence`: an empty path, an unreadable record, `slack`, `terminal`. Correct. One caveat, not a defect: a `PendingResume` stored earlier carries the presence from when it was stored.

## Findings

**F1 — MEDIUM, design.** `internal/slackengine/outbound.go:96-133`. The summary line for mail older than 24 hours is lost when a read fails after the stale mail was skipped.
- Evidence: the skip count lives in the local variable `stale` and is posted only by `reportStale()`, called at the budget return (line 112) and at the end (line 131).
- The Slack failure return (line 122) and the `writeOutbox` error return (line 128) skip it.
- Stale mail has the lowest ids, so it is handled first and recorded in `record.Handled` through `writeOutbox`.
- On the redelivered read those ids hit `continue` (line 108) and are never counted again, so the line "N messages older than 24 hours were not posted…" never posts.
- A failed `PostMessage` of that line is also only logged and the count dropped (lines 102-105).
- This contradicts D5 and its acceptance criterion: stale mail is acknowledged without a post, and one line names how many were skipped.
- Fix: keep the skipped count in the outbox record (for example `Stale int`), cleared with the record on a confirmed ack, and post the line when the read finishes, or before any early return.

No other lens-owned findings.

## Tool calls
1. `git diff` of `internal/orchengine` prompt.go, resumecontext.go, state.go and watcher.go: the manifest lacked the presence-line hook.
2. grep of `internal/state` for file modes: to check the owner-only claim.
3. grep of `internal/logger` for `EpisodeSet`: to check whether the two relay loops can share it.
4. Read of `internal/state/state.go` lines 20-130: `WriteJSON` and its lock and mkdir behaviour.
5. grep of `internal/fsx` for `AtomicWriteBytes`: temp-file mode, to settle the file permissions.
