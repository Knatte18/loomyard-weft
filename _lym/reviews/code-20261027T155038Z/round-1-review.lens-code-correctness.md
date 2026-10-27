```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness. For each function or branch the diff adds or changes, I ran one concrete case against the code in context. I found no BLOCKING or MEDIUM defects.

**List: function or branch, the case I ran, verdict**

- `typeNotice`, idle strand, notice unreserved, send lands: reserve, send, `markNoticeRead` drops the pane reservation and journals one read. Correct.
- `typeNotice`, a hook claim took the notice first: `reserveNotice` sees `reserved[id] != ""`, returns false, nothing is typed. Correct.
- `typeNotice`, send returns `ErrBusy` or another error: `releaseNotice` drops the reservation, `answerLocked` wakes the holder's waiters, the message stays unread. Correct.
- `typeNotice`, `ErrNotLanded`: treated as typed and journaled read, the same as before the diff. Correct.
- `reserveNotice`, `releaseNotice`, `markNoticeRead` against `Read`, `ClaimAtBoundary` and the pane probe: all three read paths treat `paneReserver` as reserved.
  - `unreadLocked` (pool.go:400) skips any `reserved[id] != ""`.
  - The pane probe (deliver.go:432) tests `s.reserved[id] == ""`.
  - `Ack` and `Release` delete only their own `ids`, and a reader's name never equals `"\x00pane"`.
  - A message cannot be delivered twice or lost on this path. Correct.
- `markNoticeRead`, notice forwarded or withdrawn mid-send: the reservation is dropped and the journal write is skipped by the holder and status check. Correct.
- `do`, `appended` gate: every `writer.append` call site is inside `do`.
  - The other direct `s.mu.Lock()` sites (hasPending, pendingReceivers, remainingTTL, sweepSubjects, Log, Queued) only read.
  - So each change to the unread set (post, read, forward, withdraw, return, undelivered, expire) triggers `syncFlagsLocked`, including the case where `fn` fails after an append. Correct.
- `syncFlagsLocked`, failed create or remove: the name stays out of or in `s.flags`, so the next sync retries. Correct.
- `ReconcileFlags` at daemon start: `Run` calls it right after `OpenStore`. It adopts every regular file, then removes stale ones and creates missing ones. Correct.
- `ClaimAtBoundary`, non-strand sender: refused with the way forward. Correct.
- `ClaimAtBoundary`, paused strand: returns an empty reading and reserves nothing. Correct.
- `ClaimAtBoundary`, hold owned by X, mail from X and Y: only X's mail is claimed, and skipped mail does not count against the budget. Correct.
- `ClaimAtBoundary`, a first message over 8 KiB: handed over whole (`len(reading.ids) > 0` guards the break). Correct.
- `ClaimAtBoundary`, several messages: it breaks at the first message that does not fit, so order stays oldest first and nothing later is skipped past it. Correct.
- `handOut` through `Client.exchange` with an empty claim: `AckRequired` is false, so `exchange` never calls `handle`, `hookRead` writes nothing, and no `{"additionalContext":""}` is printed. Correct.
- `handOut`, non-empty claim: `writeContext` runs before the ack is sent. A write error returns an error, the connection closes without an ack, and the daemon releases, so the mail stays unread with its flag standing. Correct.
- `hookRead`, unknown engine: the error goes to stderr with exit 1 and stdout stays empty. Correct.
- `hookRead`, daemon refusal or `ErrWaitTimeout` after 10 s: handled the same way, with the `creel: ` prefix added once. Correct.
- `creelMailHookCommand` shell loop:
  - `a:b:c` gives `a.b.c`.
  - An unset or empty `LYM_STRAND_NAME` gives `dir/` + `""`, `[ -f ]` is false, exit 0.
  - A name with no colon is unchanged.
  - `cd` failure falls through to `lym` in the current directory.
  - Correct.
- `buildSettings` and `Prepare`: `PostToolUse` is omitted when `c.dirs.Board == ""`, and the Windows path goes through `PosixPath`. Correct.
- `hookContext` refactor: `SessionStartContext` produces the same JSON. Its error text changes to `marshal SessionStart context`; I did not check whether a test pins the old text, which is the affected-set lens's area.
- `FileName` and `CreelFlagDir`: `:` becomes `.` one to one, and the path is `board/.lym/creel/flags`. Correct.

**Findings**

1. LOW, design, `internal/creelengine/flags.go:57-76` and `server.go:81`.
   - If `ReconcileFlags` fails at start (for example `MkdirAll` fails), the daemon only logs a warning and runs on.
   - Every later `syncFlagsLocked` then warns on each failed create, and the hook never sees a flag. Mail then reaches the strand only through the idle pane path.
   - A failed flag create is retried only at the next journaled change, not on a timer.
   - This is graceful degradation and matches the decision record's wording that flags are derived state, so it is probably acceptable.
2. NIT, design, `internal/creelengine/verbs.go:188-191`: the budget sums `len(MessageLine)` without the `\n` that `hookRead` joins them with. A claim can exceed 8 KiB by up to one byte per extra message. The decision record says "lines up to an 8 KiB budget", so this is only a literal-reading difference.
3. NIT, design, `internal/creelengine/flags.go:32-45`: `s.flags` is a cache of what the directory holds. A flag file deleted from outside the daemon is not recreated until the next daemon start, because `s.flags[name]` stays true. The decision record makes the daemon the only writer, so this is outside the contract.

Tool calls:
1. Read `server.go` `read`/`handOut` and `client.go` `Call`/`exchange`: my context lacked the empty-claim path and the client's ack handling.
2. Grepped `writer.append` and `s.mu.Lock` and read `verbs.go` `Read`: to check that every journal append goes through `do`, and `Read`'s reservation behaviour.
3. Read the direct-lock sites in `deliver.go`, `sweep.go` and `verbs.go`: to confirm they only read the pool.
