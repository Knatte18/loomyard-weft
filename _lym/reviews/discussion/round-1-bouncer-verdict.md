---
verdict: CONTINUE
rationale: "Round 1 carries four BLOCKING design findings (F1, F2, F3, F4) and four MEDIUM design findings (F5, F6, F7, F8); the decision rule gates on both, and the review has no Focus departures section to ratify or reject."
---

No finding is relabelled, so there are no departed findings to name.

The four BLOCKING findings each show a real dead end in the design:
- F1: a re-run of `lym adopt` holds the fix task and deadlocks the fix loop.
- F2: held runs fill `max_runs`, so the fix task never starts.
- F3: cancelling the fix task leaves a draft front entry that wedges the worktree.
- F4: `landed` can never fire, because Land prunes the done entry.

The MEDIUM design findings F5 to F8 are slug collisions, the task-guard relation, the `upsert`-only refusal and the `on_land` loss across a dobby restart.
F9 is a MEDIUM `consistency` finding and F10 a LOW `design` finding, so neither gates.

F1 to F4 and F6 to F8 cite the declared `PATTERN-front-task` and `PATTERN-adopt-hold` behaviour or the record's own Decisions, and none is downgraded.
The review is the first round, so no ledger key recurs and none was open earlier.
