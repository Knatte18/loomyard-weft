```yaml
lens: fix-affected-set
outcome: reported
```

F1 (the only finding; its premise is the stop cases of `sessionStopWanted`): no findings.

The fix covers every stop case, and each of the four now has a row that fails without it.

| Case | Row that pins it |
|---|---|
| Merged | `MergedStopsTheSessionBeforeTheReReadOfTheTree` (`prreview_wait_test.go:429`, liveness at each read plus counts 1,1) |
| `rec.ReviewEnded` | `ReviewEndedOnAnOpenPullRequestOnADirtyTreeHaltsNamingTheFiles` (counts 0,1). The dirty tree means the Continue case is false, so only the `ReviewEnded` case can stop it. |
| Continue at a clean tree and the gated head | `ContinueOnAClosedPullRequestAtTheGatedHeadIsDoneAndRecordsIt` (counts 1,1). `ReviewEnded` is unset when the wake reads the record, and a closed PR is not an approval, so only the Continue case stops it. |
| Approval | `ApprovalOnADirtyTreeStopsTheSessionBeforeTheReReadAndResumesItWhileWaiting` (2,1), and the cancel row at line 603 |

The Continue-ungated, Continue-dirty and Continue-dirty-closed rows still pin "no stop", which is the narrowing.

A closed-PR row that starts `ReviewEnded` on a dirty tree is the only other row that could stop for a recorded ending. It has no live session and no assertion, so it adds nothing here. It needs nothing for F1, because the open-PR row already pins the case.

Calls made:
1. `git diff HEAD~1 -- prreview_wait_test.go` plus a grep for `assertSessionCounts` and `sessionStopWanted` across the landingshed tests. I needed the committed diff and the set of existing stop assertions, which my context lacked.
