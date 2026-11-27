```yaml
lens: fix-way-forward
outcome: reported
```

Checked the ways forward the fix introduced in the decision record and support log.

List of ways forward checked:
- D7: the `resetUnnamedTip` refusal says to merge origin's branch through the task worktree, then run `lym fabric push`. This can be carried out: it matches today's `codeDivergedWayForward`, and a push that lands clears the record (pushclassified.go:474).
- D7: the takeover stop uses the same merge followed by `lym fabric push`. This can be carried out: the merge makes origin's commits reachable, so the push fast-forwards.
- D3: clearing the refused-push record through its way forward removes the refused mark. This holds for pair and records sides, but not for the prime's code branch (finding 1).
- D5: a refused landing push makes the pull-request check a blocked halt that names the refusal's way forward. That way forward is not reachable for the prime's code branch (finding 1).
- D5: a held `done` or records commit is released when the parent push lands, through push-now marks. This works only while the parent push can land (finding 1).
- D5: the resume after a crash records the owed tip. The state it names is reachable, and the resume makes progress.
- D6: a dead driver is reported once as a stop. This makes progress.
- D9: the scratch repository with `git check-ignore --no-index`. This can be carried out.
- D13: with pushes off, the pair is removed as today. This can be carried out.

Findings:

1. MEDIUM. Changed text: D3, "Clearing that record through its way forward removes the mark and lets the pusher start again". It also reaches D5's "owes a parent push while its ledger entry stands, refused or not", D5's held-commit rule and D13's landed-pair wait.
   Summary: for a refused landing push on the prime's code branch, the record's way forward clears neither the record nor the mark. The landing entry then stands for good, which blocks every later board push, the run's records push, D13's teardown and every pull-request landing on that parent.
   Evidence: `FailedPushWayForward` names `git push` for the prime's code side. `pushStep` returns "`git push` in %s" for `RepoCode && Prime` (failedpush.go:255-256), and `codeDivergedWayForward` does the same when `prime` is set (failedpush.go:317-318). A plain `git push` runs no lym path. `ClearPushFailure` is called only from pushclassified.go:474, remove.go:305 and originreconcileended.go:210. So the record stands, `RefusedPushStands` stays true (pusherreads.go:153), and the refused mark is never removed. The fix needs a way forward for the landing target's refusal that clears it: a lym verb, or the pusher noticing that origin's parent now holds the owed tip and dropping the entry.

Tool calls:
- grep for `RefusedPushStands` and the failed-push clear functions, to find how a refused record clears, which my context did not show.
- `sed` of pusherreads.go:140-175 and failedpush.go:255-300, to read the refusal record's way-forward text, which my context did not show.
- grep for `ClearPushFailure` callers and `codeDivergedWayForward`, to confirm that no non-lym push clears the record and to read the prime code-side way forward.
