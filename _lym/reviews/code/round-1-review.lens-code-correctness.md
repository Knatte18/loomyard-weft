```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens on the `c15d6390..HEAD` diff. I ran each added or changed branch from the code in context.

**Per-branch list**

| Function or branch | Case run | Result |
|---|---|---|
| `ringHook`, marked waiter, no pause | Waiter registered, set unchanged, `claimLocked` over the claimable set | Correct. Reading reserved, `answerLocked` delivers it, mark set, Stop count reset (`verbs.go` `ringHook`). |
| `ringHook`, empty claim | `ids` empty | Unreachable. `wakeHookStrand` returns on `len(ids)==0` and `ringHook` requires `current == ids`, so the missing explicit guard is not a defect (`deliver.go:~257`). |
| `ringHook`, marked waiter registers after the pause read | `pauseRead` false, `anyTakesMail` true in the locked step | Correct. Returns nil, nothing rings, waiter stays registered, and `HookWait`'s `wakeLocked` makes the worker probe again. |
| `ringHook`, pause read errors or finds a pause | `Paused` returns err or true | Correct. Returns false with a Debug "paused" line, waiter stays. |
| `ringHook`, unmarked waiter | `takesMail` false | Correct. Plain `OutcomeRing`, no claim, no pause read. |
| `awaitOutcome`, ring and client already gone | Outcome in the channel, `watcher.closed` closed | Correct. `watcher.stop()` is idempotent, the write is ignored, `awaitAck`'s `readLine` fails, `Release` runs. |
| `awaitOutcome`, select picks `closed` or `done` with the outcome still queued | Ring outcome unread in the channel | Correct. Deferred `Waiter.Cancel` unlocks first, drains the channel and calls `Reading.Release` outside the lock (`pool.go` `Cancel`). |
| `awaitOutcome`, daemon stops after the outcome was taken | `awaitAck` waits | Correct. The connection closes, `readLine` errors, `Release` runs. |
| `Reading.Ack`, ring, all ids journaled | 2 ids | Correct. Both journaled, one echo item each, flag synced via the deferred sync, ring marked answered, `acknowledged` logged. |
| `Reading.Ack`, ring, first write fails | `writeFault` on id 0 | Correct. `released = ids[0:]`, all reserved ids deleted, no echo item, mark stays unanswered, `ack-failed` Warn lists no echo ids, error returned, `r.ids=nil`. |
| `Reading.Ack`, ring, second write fails | id 0 journaled, id 1 fails | Correct. Echo holds id 0 only, id 1 and later are released. |
| `Reading.Ack`, ring, id no longer the holder's unread mail | `msg.Holder` changed | Correct. Reservation deleted, echoed, no journal write. |
| `Reading.Ack`, claim that handed out an echo, journal fails | Echo plus mail | Correct. Echo dropped by `settleEchoLocked(true)` before the loop, `dropped` logged, mail ids released, the holder's mail waiters are answered, deferred sync runs on the error return. |
| `Reading.Ack`, read's reading, journal fails | `r.read` true | Correct. Ids stay reserved as before, no `logOutcome` (empty path), the old Warn is kept in `awaitAck` for reads only. |
| `Reading.Ack`, echo-only claim | `ids` empty, `echoIDs` non-empty | Correct. Echo dropped, `acknowledged` logged, no ring mark or Stop count touched. |
| `Reading.Release`, claim with echo and ids | Echo handed out, never acknowledged | Correct. Echo unreserved and still owed, flag synced, ids released, waiters answered, `released` logged once. |
| `Reading.Release` after a successful Ack | `ids` nil, echo settled | Correct. `held` false, no second log. |
| `ClaimAtBoundary`, paused with an echo owed | Pause covers the strand | Correct. Echo claimed before the pause check, mail withheld, no Stop-count reset, no Debug line because an echo was handed out. |
| `ClaimAtBoundary`, Stop at cap or count unknown with an echo | `stop-cap` or `stop-count-unknown` | Correct. Echo handed out, mail withheld, count not incremented. |
| `ClaimAtBoundary`, nothing at all | No ids, no echo | Correct. One Debug line, with reason `held` or `nothing-claimable` from `nothingClaimableReasonLocked`. |
| `claimEchoLocked` and `claimLocked` budget | Echo near 8 KiB, then a message | Correct. The first item is always taken, the join newline is counted (`size++`), the echo is counted in `used` before messages, and an item that does not fit stays owed. |
| `flaggedFilesLocked` | Echo owed, no unread mail | Correct. Flag stands for the echo; `Ack` and `Release` sync flags. |
| `HookOutput`, Stop with mail and no echo | `mail` set, `echo` empty | Correct. `systemMessage` omitted, no `decision` or `reason` (`capabilities.go`). |
| `HookOutput`, Stop with echo and mail | Both set | Correct. `systemMessage = echo`, `additionalContext = mail`. |
| `HookOutput`, PostToolUse with echo and mail | Both set | Correct. `systemMessage = echo + "\n" + mail`. |
| `HookOutput`, echo only, either event | `mail` empty | Correct. Only `systemMessage`; `hookSpecificOutput` omitted. |
| `hookRead` and `hookReadExit` | Stop with mail, echo-only Stop, nothing claimed, write failure | Correct. Exits 0, 1, 1 and 1; mail printed twice never occurs. `wrote = mail != ""` is set only after a successful write; a non-AckRequired response never reaches the handler. |
| `hookWait` and `hookWaitExit` | Ring with lines, supersede, older daemon with no mail, stderr write fails, ack unsent, confirmation refused or lost | Correct. Exits 2, 0, 0, 0, 0 and 2. `ErrAckUnsent` is found through `connLostError.Unwrap` and the multi-`%w` wrap. |
| `Client.CallWatching` | Connection lost after `state.handled` is set, hook-wait | Correct. Returns the error before the reconnect path. A connection lost before the ring still resumes with the same request, `TakesMail` included, because `resume` returns `req` unchanged. |
| `Client.exchange` | Handler fails, ack write fails | Correct. A handler error comes back as a plain error and the connection closes, so the daemon releases. An ack-write failure comes back as `connLostError` wrapping `ErrAckUnsent`. |
| `typeBell` logging | Busy send repeated each probe | See finding below. |

**Findings**

1. **LOW, design — `internal/creelengine/deliver.go`, `typeBell`.**
   - Every `ErrBusy` send logs an Info `creel bell` line with outcome `busy`.
   - A busy send records nothing, so the next probe tries again and logs another Info line, one per probe tick while the pane stays busy.
   - D5 asks for one line per bell send and puts a poll's repeated empty outcome at Debug (`nothing handed out` → Debug); a retried busy send is the same kind of repeat.
   - This is a judgment call; D5's text can also be read to want Info for every send.
   - Evidence: the `typeBell` diff, `deliver.go` around lines 294–320.

No other defect found in the changed code.

**Tool calls:** none. Everything I needed was in the inherited context.
