```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each proposal the fixer carried out as proposed and found three cases where the proposal breaks something, plus one smaller one.

**BLOCKING**: the F18 proposal undoes the F2 fix.
- **Changed text:** D7's second replay bullet ("rewrites a replayed commit's `Warp-SHA` trailer … only when every commit origin added is records-only; otherwise it stops as owed, naming origin's code commits"). It also touches D5's new leveling rule ("then replays the local records-only commits onto origin's tip as D7 replays them") and D8 ("brings the local parent level … by D5's rule").
- **Case:**
  - GitHub squash-merges the view, so origin's parent tip gains a code commit.
  - The local parent holds unpushed records-only commits, as the review's F2 premise says it always does.
  - D5's leveling replays "as D7 replays them". D7 now stops as owed whenever origin added a code commit.
  - So every pull-request landing with unpushed parent records halts. This contradicts the new D8 criterion "with uncommitted and unpushed parent records, a pull-request landing and a local landing both complete".
  - The same stop hits every pull of a task branch after another machine pushed code.
  - F18's premise is only that the trailer must not name an unjudged tip. Replaying without rewriting the trailer, or keeping the old value, settles that without stopping.

**MEDIUM**: the F4 pick "not reachable from any origin ref" lets the prime guard push the operator's code.
- **Changed text:** D7, "A commit origin lacks is a commit not reachable from any origin ref."
- **Case:**
  - The operator pushes `origin/feature`, then in the prime runs `git merge --ff-only feature` on `main` and does not push `main`.
  - The next records commit runs the prime guard. Every commit not reachable from an origin ref is records-only, so the guard pushes `main`.
  - That carries the operator's feature commits onto `origin/main`, which the guard exists to prevent.
  - Reading reachability from `origin/&lt;prime branch&gt;` alone would close this. A landing merge's second-parent commits are then handled by the "merge commit fabric wrote" admission.

**MEDIUM**: the F20 proposal picks `deferred`, a class that never sends its notice.
- **Changed text:** D7, "its failed-push record has class `deferred` … Its notice names the operator's unpushed commits and `git push` as the way forward".
- **Evidence:**
  - `internal/fabricengine/failedpush.go:170` reads `if f.Notified || f.Class == PushFailureDeferred { return f, nil }`, so a deferred record sends no creel notice.
  - The notice line is built from `FailedPushWayForward(f)`, which is chosen by class, not by the operator's commits.
  - `failedpush.go:38` defines `deferred` as "held back, not refused, and is retried by the next push of the branch".
- **Result:** the prime run waits at its transition and the operator is never told. A guard refusal needs its own class, which `FailedPushesOwedBy` must also read as owed (`failedpush.go:284`), or the notice rule has to change for it.

**MEDIUM**: the F1 idempotent publish stops recreating a view ref that origin lost.
- **Changed text:** D8, "It reuses the existing `lym-view/&lt;branch&gt;` tip when that tip's tree and parent equal the ones it would write", and the D8 criterion "push nothing the second time".
- **Case:**
  - Someone deletes `lym-view/&lt;branch&gt;` on origin while the pull request is open, or before Finalize's `Reopen` (`finalize_pr.go`, `landing.reopen`).
  - The local view ref still matches, so the publish pushes nothing and the pull request's head stays gone.
  - The review's Rejected section relies on "a view ref deleted by someone else is … recreated by the next publish". F1's wording, read against the local ref, breaks that.
  - The reuse test should compare against origin's view ref, or push when origin lacks it.

**LOW**: the F12 refusal stalls every prime run while the operator works on another branch.
- **Changed text:** D7, "Fabric refuses to commit records in the prime while it is not on that branch".
- **Case:** the operator checks out a feature branch in the prime for an hour. Every prime run's transition commit refuses, and the only way forward is to switch back. No acceptance row or bound states that runs stall for that time. It needs a stated bound, or the docs warning D2 requires.

Further tool calls:
1. Read `internal/fabricengine/failedpush.go:24-44` and grepped for `PushFailureDeferred`: I needed what the `deferred` class means and who reads it, and my context lacked that.
2. Read `internal/fabricengine/failedpush.go:158-180`: I needed to confirm whether a `deferred` record sends its notice, and my context lacked that.
