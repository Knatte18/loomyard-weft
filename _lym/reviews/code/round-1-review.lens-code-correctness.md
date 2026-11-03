```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, covering each function or branch the diff changes, one concrete case run per item.

**Per-branch list**

- `pollEventsTick` gated branch, files present, shell inside the bound: correct. The nudge is queued and `expiredTurnEnd` returns nothing. `waitingTasks` and `waitingTurnEnd` stay set, `countedTasks` was cleared earlier, and the next empty-data tick reaches `expiredTurnEnd` again.
- `pollEventsTick` gated branch, files missing: correct. It holds at once as before. `waitingTasks` is cleared and `countedTasks` is set, so no stale list leaks.
- `pollEventsTick` gated branch, a turn start past the waiting turn end in the same batch: nudge skipped, correct. `waitingTasks` stays set, which is the problem in F1.
- `expiredTurnEnd`, gated with files missing: returns nothing for a non-empty `waitingTasks`. That is correct when the list holds a fork or an awaited shell, and unreachable in practice for a shell-only list, because the branch above has already cleared it.
- `expiredTurnEnd`, gated with files present, shell at exactly the bound: `now.Sub(first) < bound` is false, so it returns Done. `countedTasks` is set and `waitingTasks` cleared, so there is no repeat Done. Correct.
- `expiredTurnEnd`, ungated paths: unchanged in behaviour after the refactor. The `waitingTurnEnd == nil` guard is kept.
- `shellFirstSeen` stamping: `recordWaiting` stamps before any `expiredTurnEnd` read in the same tick, and Attach starts with an empty map, so the first sighting is stamped. A zero time cannot occur. The bound is measured from first sighting, not from the turn end. Correct.
- `unawaitedShellsPastBound`, staggered shells A and B: B's first-seen time is now, so it returns false until B has aged the bound. Correct.
- `queueShellNudge`, two turn ends over the same shell: the second queues nothing. A new shell is named alone. Correct. Shells with an empty `ID` share one key, which is existing keying and not new.
- `sendShellNudge`, run covered by a pause: dropped. The shells were already marked named in `queueShellNudge`, so the agent is never told. See F2.
- `sendShellNudge`, send fails (`ErrSessionBusy`, `ErrSubmissionNotLanded`, `ErrPaused`): logged at Info, no retry, and the shells stay marked named. Same effect as F2, as documented. Nothing swallowed beyond that. `sendWithin` takes `sendMu` and uses `startHold`, and the nudge runs before `settleDeferredHold` in the same tick, so there is no deadlock.
- `sendShellNudge`, send blocks: `sendVerified` can wait up to `send_ready_timeout_s` while the loop is single-threaded, which delays liveness and pause ticks. The pane is normally idle at a turn end, so this is bounded and minor. Not a finding.
- Done returned from `expiredTurnEnd` in `Wait`: it is routed as a gated Done to `handleGatedBoundary`. `waitingTasks` is nil after Done, so it is not re-fired. Correct.

**Findings**

- **F1, MEDIUM, design**: `wait.go:1112-1129`, `wait.go:881-883` and `901-903`. The gated expiry can count Done on a stale `waitingTasks` list after the shell has completed.
  - Trigger: a turn start with no event (`len(events)==0` returns `expiredTurnEnd()` without clearing `waitingTasks`) falls into the no-event branch while the list still holds a shell.
  - Concrete case: the shell finishes after 10 min 30 s of a 10 min bound and its completion starts the agent's next turn. Until that turn ends, `waitingTasks` still lists the shell, whose first-seen time is past the bound. The next tick returns `OutcomeDone` while the agent is mid-turn reading the shell's result.
  - Before this diff a gated shell-only waiting turn end was Done at once, so the window did not exist. The only guard is `boundaryIdle`, which applies only to engines with the `SessionCycler` idle reading.
  - Fix: in `pollEventsTick`, when `turnStarts` reads a turn start past `run.waitingTurnEnd.offset`, clear `waitingTasks` and `waitingTurnEnd` (keep `countedTasks` semantics) so the expiry cannot fire in a turn already under way. Or have the gated branch of `expiredTurnEnd` refuse Done while an unmatched turn start follows the waiting turn end.
  - Test: no test covers a turn start after the waiting turn end with a bound crossing.
- **F2, LOW, design**: `wait.go:1160-1175` and `1186-1201`. Shells are marked named in `queueShellNudge` before the send. A pause drop or a failed send leaves them marked, so a later waiting turn end over the same shells never nudges. The run then reaches Done at the bound with the agent never told.
  - Fix: mark shells named only after `sendWithin` succeeds (record the pending IDs with the queued text). Or, if the drop is intended, state it in the `sendShellNudge` doc comment, which currently covers only the failed send.

**Tool calls:** none. The `wait.go`, `send.go`, `hold.go`, `waitmark.go` and diff reads already in context were enough.

**Outside my lens, noted only:** the `waitmark.go:175-177` comment on `shellWaitActive` ("a gated run's shell-only waiting turn end is judged at once") is now stale.
