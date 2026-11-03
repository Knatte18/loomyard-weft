```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F7) and build the concrete case where one is wrong.

**Proposals I could not break:**
- F1: clearing `waitingTasks` and `waitingTurnEnd` on a turn start past the waiting offset. A fork or shell that is still outstanding is re-listed by the next waiting turn end, so no case loses work. The `len(run.waitingTasks) > 0` guard keeps an ungated hold's `countedTasks` from being overwritten with nil.
- F4: marking shells named only after a successful send. The line is re-queued only at a waiting turn end, so a persistently failing pane costs one send attempt per waiting turn end and no more.
- F5, F6, F7: I found no unqualified claim in the new wording that the code contradicts.

**Findings:**

1. LOW — `internal/shuttleengine/wait.go`, `pollEventsTick` clear and `recordWaiting`.
   - Summary: the shell's first-seen stamp survives the F1 clear, so the agent gets no new wait window after the nudge.
   - Case: the shell is first seen at t0 and the nudge is sent. The agent starts a turn, reads, and ends its turn again over the same shell at t0+bound+ε. `recordWaiting` keeps the old stamp, and the gated branch (files present) goes to `expiredTurnEnd`. `unawaitedShellsPastBound` is already true, so it returns Done in that tick. The agent that was told to wait and read the result is counted done at its first re-end. This is probably what the reviewer intended, since the bound is measured from first sighting. The run still never gets a full bound after the nudge, and the reviewer's stated aim was "a run that waits for the shell".
   - Evidence: `recordWaiting` stamps `shellFirstSeen` only for unseen ids, and the F1 clear does not reset the stamps.

2. LOW — `expiredTurnEnd` doc, `wait.go`.
   - Summary: the doc still says only counting or expiring clears the waiting list.
   - Evidence: "Counting or expiring a turn end clears the waiting list and keeps its tasks in countedTasks for the run's record." The F1 change adds a third clearing path, a turn start in `pollEventsTick`. That path lives outside `expiredTurnEnd`, so the doc is not false, but the `pollEventsTick` doc is now its only mention.

Tool calls made: none; my context already held every file I needed.
