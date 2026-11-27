# Card 18 — reset-hand-commit-guard

**Edit:**
- `internal/fabricengine#pushJob.resetUnnamedTip`
- `internal/fabricengine#codeDivergedWayForward`
- `internal/fabricengine#FailedPushWayForward`

**Create:**
- `plan:internal/fabricengine#unmadeOriginCommit` -> `func unmadeOriginCommit(path, named, originTip, branch string) (string, error)`
- `plan:internal/fabricengine_test#TestRejectedCodePush_RefusesACommitThisMachineDidNotMake` -> `func TestRejectedCodePush_RefusesACommitThisMachineDidNotMake(t *testing.T)`

**Uses:**
- `internal/fabricengine/doc.go`
- `internal/fabricengine#pushJob.repushOverOriginContained`

**Intent:** D7: no push resets origin's code branch past a commit lym did not make on this machine.

`resetUnnamedTip` calls `unmadeOriginCommit` before its leased reset.
It lists the commits origin's tip holds past the named tip and returns the first that neither a local branch (`refs/heads/*`) nor an entry of the local code branch's reflog reaches;
remote-tracking refs and their reflogs do not count, since every fetch fills them with origin's commits.
With such a commit the reset is not made: the push is refused as `not_fast_forward`, its record's `Commits` naming that commit,
and `codeDivergedWayForward` names it with the way forward: merge origin's branch into the task branch through the task worktree, then run `lym fabric push`.
`FailedPushWayForward` passes the commits through. A commit every one of whose ancestors past the named tip is reached resets as today.
The rule sits in `resetUnnamedTip`, which every path the change description of `pr-modes-no-agent` names reaches; the hand-over's and destroy's moves of an origin branch need none, per D7.
After a takeover the new owner lacks the old owner's local refs, so its reset is refused and named, and the same merge and `lym fabric push` is its way forward.
Bound: the rule relies on git keeping reflog entries for `gc.reflogExpireUnreachable` and `gc.reflogExpire` (git-gc(1)); a hand commit that entered the reflog without the branch keeping it passes, and a commit of lym's own whose entry expired is refused and named.

`TestRejectedCodePush_RefusesACommitThisMachineDidNotMake`, tier1, red-first, over a pair fixture with a second clone:
a commit pushed to the task branch from the second clone, then a local code commit and a pair push, is refused naming that commit and the way forward, with origin's branch left at it, where the merge base resets past it;
a commit held only by the local code branch's reflog, the branch reset off it, counts as made here and the reset goes as today.

**ImpactSummary:** Adds one guard inside the code side's rejected-push retry and one clause to its way forward; every other push path is unchanged.
