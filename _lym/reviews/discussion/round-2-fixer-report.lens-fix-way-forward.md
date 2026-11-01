```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward this fix added (refusals, notices, next steps, criteria) for whether each can be carried out from the state it names and leads to progress.

**MEDIUM: the front-task refusal names a first step that refuses**
- **Changed text:** D8's new bullet: "`boardengine` refuses a change of project or kind on an entry whose stored `front` is true … and as its way forward its removal with `lym batten cancel <slug> --remove`, then `lym board remove <slug>`, and a re-run of the command that filed it".
- **Summary:** A released front task that never started is the most likely target of a hand upsert or rename. For that task, the named first step fails with cancel's nothing-left refusal. That refusal's own way forward is `check it with "lym board list"`, so the operator gets no route to `lym board remove`.
- **Evidence:** `internal/battencli/cancel.go` `refuseNothingLeft` returns `nothing of %s is left to cancel; way forward: check it with "lym board list"` when there is no record, run status, session, process, marker or pair part. The fix's own D8 removal bullet treats this refusal as "the first step done" only inside `fronttask`. The hand way forward gets no such allowance.
- **Also:** After `lym board remove`, the entry reads gone, so D8's "An entry removed by hand … reads gone and its action runs; for `adopt-check` that is one more check" fires before the named re-run. The sequence still makes progress but starts an extra check.

**LOW: the new notice starts a second check alongside the hand re-run**
- **Changed text:** D6's new bullet: "The filing does not go ahead while a removal it needed failed; its notice carries the failed step's error and names `lym board remove <slug>`, then a re-run of `lym adopt <subject>`". The same applies to D5's changed "a failed removal carries the failed step's error and names `lym board remove <slug>`".
- **Summary:** When the cancel step fails, D8 says the record stays ("a failed cancel leaves the record"). The hand `lym board remove` then makes the entry read gone, and on the owner machine dobby runs `adopt-check`. That check does not reset the try count and may file or refuse at the limit. It runs next to the check the operator's `lym adopt <subject>` re-run starts, and the notice says nothing about this ordering.
- **Evidence:** D8: "a failed cancel leaves the record, so the task's later removal by hand still runs its action once". D5: "a check started by a fix's landing does not" reset the count. `Store.RemoveTask` (`internal/boardengine/store.go:591`) refuses nothing but an absent entry, so the board removal goes through.

The other new ways forward are reachable and make progress:
- the not-owner notice of `lym adopt check` (names `lym adopt <subject>` here)
- the halt reason from the second marker (re-run `lym adopt`, whose red check now removes the halted task)
- the healing of a draft left by a dying filing (the next check removes it)
- the new D6, D7 and D8 acceptance criteria

Tool calls:
1. Grepped `internal/boardengine` for `RemoveTask` and a run-held refusal, because my context did not show whether `lym board remove` refuses a run-held entry.
2. Read `internal/boardengine/store.go:580-625` for `RemoveTask`'s refusals, for the same reason.
