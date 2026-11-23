```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the diff's changed functions and branches.

**List: each function or branch, the case run, and the result**

`ReadPRReviewStanding` (`internal/landingshed/prreviewask.go:107-134`):
- `ReadParentAsks` fails: the error comes back as `"", err`. Correct.
- The record is empty or has no file: the loop does not run, `LatestTagged` finds nothing, and the result is ask-again. Correct.
- Satisfied done at h, then an open ask at h before its deadline: the loop skips the open ask on state and finds the satisfied report. Result: satisfied. Correct, as the entry intends.
- Satisfied done at h, then undelivered, no-parent and timeout asks at h: the loop skips all three. Result: satisfied. Correct.
- Satisfied done at h, then an open ask at h past its deadline (`now == Deadline`): the loop finds the satisfied report. Result: satisfied. Correct.
- Satisfied done at h, then a done ask at h with findings: the loop reaches the findings ask first and hits `break`. The latest ask is done, which is neither pending nor open. Result: ask-again. Correct.
- Done with findings at h, then an open ask at h before its deadline: the loop skips the open ask and breaks on the findings. The latest ask is open and `now.Before(deadline)` holds. Result: awaiting. Correct.
- Done with findings at h, then an open ask at h with `now == deadline`: `Before` is false. Result: ask-again. Correct, and the boundary is unchanged.
- Satisfied at an old head, then an undelivered ask at h: the loop skips the old ask on digest. The latest ask is at h and ended. Result: ask-again. Correct.
- Satisfied done at h, then a newer Code-Review-tagged ask: the loop skips the other tag. Result: satisfied. Correct.
- A cleared satisfied ask at h is the only done ask: the loop skips it on `Cleared`. The latest ask decides. Correct per the doc. In production, `ClearParentAsk` has a driftengine caller only, so a cleared PR-Review ask does not arise.
- `ReadParentReport` fails on the latest done ask at h, for example a missing report file: the error comes back. It is neither swallowed nor misreported. Correct.
- With no done ask at h, the fallthrough gives the same result as the old switch in every state: pending or open before deadline, open past deadline, timeout, no-parent, undelivered, not-sent, cleared, and another head. Correct.

`parentReviewAskVerb` (`internal/loomcli/parentreview.go:326-336`), reached through the changed standing:
- Satisfied done at h, then ask N open at h before its deadline, then `parent-review ask` runs: the standing is satisfied, not awaiting, so the refusal is skipped and `askPR` records and posts ask N+1. Incorrect against the verb's contract: see F1.

`prContinueUnderAutonomy` (`internal/loomcli/prcontinue.go:102-131`) and `judgeParentAnswer` (`internal/landingshed/prreview.go:1012-1030`):
- Satisfied done at h, then a failed or open ask at h: both see satisfied and end the review. Correct, as the entry intends.

**Findings**

F1
- severity: LOW
- class: design
- location: `internal/landingshed/prreviewask.go:121-122`, reached in `internal/loomcli/parentreview.go:330`
- summary: When a satisfied answer stands at h and a later ask at h is still open before its deadline, `ReadPRReviewStanding` returns satisfied. `parentReviewAskVerb` refuses only on awaiting, so it records and posts a second ask while the earlier one still awaits its answer. That breaks the verb's documented "It refuses an ask still awaiting its answer at the head" (`internal/loomcli/parentreview.go:300`). The parent's answer to the open ask is then refused as not-latest (`internal/burlerengine/parentreview.go:375-377`), and that ask is left open.
- evidence: `internal/loomcli/parentreview.go:330`; `internal/landingshed/prreviewask.go:112-123`; `internal/burlerengine/parentreview.go:375-377`

Tool calls: none.
