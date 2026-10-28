```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each added or changed function with concrete cases against the code in my context.

**Per function and case**

- `ClaimAtBoundary` Stop branch, with a fresh daemon and no map entry: the count is unknown, so nothing is claimed and the turn end is recorded. Correct.
- `ClaimAtBoundary` Stop branch, with `stop_hook_active` false and a count of 6: the reset to 0 runs before the cap check, so the claim proceeds. Correct.
- `ClaimAtBoundary` Stop branch, with `stop_hook_active` true and a count of 6: the claim returns empty. Correct.
- `ClaimAtBoundary` tool-boundary claim that takes nothing: the count is not reset. Correct, because the count errs early.
- `ClaimAtBoundary` budget loop, with an old-binary line of 9,000 UTF-16 units: the line is claimed alone when first, and stops the loop when it is not first. Correct.
- `ClaimAtBoundary` budget loop, with an old-binary line over 10,000 units: `claimableLocked` skips it, `flaggedFilesLocked` ignores it, and it stays for `lym creel read`. Correct.
- `LineFits` and `validateLine`, with a body at exactly the budget: measured with the widest int64 id, so it never over-admits. Correct.
- `Reply`, with a reply to EXTERNAL over 8 KiB: it keeps the 64 KiB cap. A reply to a strand or program is capped by line size, using `sender.Name`. Correct.
- `post` / `interruptBeforePost`, with an invalid body: no key is sent. With an unmarked receiver: refused before `interruptTarget`. With a hold: `interruptTarget` returns an `*interruptRefusal`, which is reworded and stores nothing. With no error: `errors.As` fails and `(interrupted, nil)` is returned. With a store failure after a sent key: `storedAfterInterrupt`. Correct.
- `post`, when `interruptTarget` returns `(true, err)` (key sent, clear failed): the error says "the post was not stored", which is accurate because the store is never reached. Correct.
- `HookWait`, with a non-strand sender: refused with a way forward. With a second waiter for the same strand: the older one is answered `OutcomeSuperseded` first, and the older waiter's later `Cancel` only stamps `waiterEnds` while the newer waiter stands. Correct.
- `wakeHookStrand`, with an empty claimable set: returns false. Same set already rung: no re-ring, goes to the bell path. Ring unanswered and the pane reads a turn: marked answered. Ring unanswered for under 30 s: the bell waits. A waiter registered with no ring: no bell. Correct.
- `wakeHookStrand`, with `TurnEndedAt` not after `mark.at` (a woken turn still running per the hook reading): no ring. Correct.
- `ringHook`, with a set changed between read and ring: no ring. After a ring: the stop count is reset to 0. Correct.
- `typeBell`, with a busy pane or a draft: nothing is recorded, so it retries. With `ErrNotLanded`: the set is recorded and never retyped. Correct.
- `probe`: `CreelHooks` goes to `wakeHookStrand`, `!PaneMail` returns `(false, true)`, otherwise notice and reminder. Correct per the amendment. See F2 for the pending flag.
- `hookReadExit`: Stop with a write returns 0 and prints any later error. Stop with no write returns 1. Tool-boundary returns 0, or 1 on an error. Subagent Stop input returns 1 with nothing printed, so the turn end is recorded. Correct.
- `hookRead`, with unparseable stdin: exit 1 and the turn end is recorded. With an event that is neither tool-boundary nor Stop: refused. Correct.
- `hookWait`, with a subagent, an unknown engine, a bad payload, an unreachable daemon, a reconnect that runs out, a superseded waiter or a stderr write failure: all return 0. A ring returns 2. Correct.
- `Client.resume`, with `VerbHookWait`: the same request is re-sent. Correct.
- `Claude.HookInput`, with an empty payload or a non-object: error. With `agent_id` set: `Subagent`. With an unknown event name: empty `Event`. Correct.
- `creelStopHookCommand`: `in=$(cat)`, then the flag test, then `printf | hook-read && exit 0`; otherwise the stamp and payload are recorded and the exit status is the append's. The flag-absent and hook-read-failed paths both record. Correct.
- `appendPromptText` and the launch and resume lines: one `--append-system-prompt-file`. The deny notice, a space, then the sentence; the sentence alone when there is no notice. Correct.
- `noticeFits`, `decisionVerb` precheck and `notifyLiveRuns` split: the precheck runs before anything is recorded, with the inline (longest) form. A multi-entry notice over budget is split into single-entry lines, each already known to fit. Correct.

**Findings**

- **F1, LOW, design.** `internal/loomcli/decision.go`, the `preRun` hunk at diff lines 373–384.
  - `preRun` now calls `hubgeom.ReedGeometry` and `agentname.FormatProgram` eagerly and aborts the whole `lym loom decision add` with exit 1 if either fails.
  - Before the diff, `ReedGeometry` ran lazily, only inside the `arbiterStrand` closure for a named caller.
  - A geometry or program-name failure now blocks recording a decision that needs no notice, such as a call with no live run or a program caller.
  - Evidence: the `preRun` hunk at diff lines 373–384 and the `arbiterStrand` closure, which still calls `ReedGeometry` itself.
  - Fix: build `programName` only when `notifyLiveRuns` needs it, or tolerate the error there.
- **F2, LOW, design.** `internal/creelengine/deliver.go:61-66`.
  - `probe` returns `pending=true` for a strand with no `CreelHooks` mark and no `PaneMail` (a Claude strand launched before the deploy, or a `lym shuttle open` session), and for a hook strand whose set is already rung or belled.
  - The worker therefore keeps probing such a receiver at the backoff ceiling for as long as mail stands unread.
  - It is bounded by the 8 s ceiling and delivers nothing.
  - Evidence: the `probe` hunk at diff lines 61–66 and `wakeHookStrand` returning false for an already-belled set.
  - Fix: return `pending=false` when the strand has neither mark.

I found no other defect in the cases I ran.

**Tool calls (1)**
- `grep` of `handOut` and `ErrNoRunningTurn` in `internal/creelengine/server.go` and `verbs.go`. `handOut` was not in the diff context. It shows that a Stop claim with no messages sends `AckRequired` false and no block.
