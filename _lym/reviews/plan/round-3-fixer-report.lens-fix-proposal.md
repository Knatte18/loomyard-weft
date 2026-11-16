```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out (F1, F2, F3) and look for cases where the proposed change is wrong.

**MEDIUM — card 12, `ReturnToPreviousTransition` paragraph**
Changed text: "commits it through `CommitAnchoredPathsWithTrailers` with the same `Run-Transition` trailer, so its `Warp-SHA` names the code tip that transition names."

- **Summary:** the F1 proposal sends the give-up commit through the trailer commit. That commit stamps `Warp-SHA` with the owner's local warp HEAD, not the previous transition's tip. So the claim does not hold, and the give-up commit can become a newest `Run-Transition` commit naming the wrong tip.
- **Evidence:** `messageWithTrailers` (`internal/fabricengine/trailercommit.go`) sets `warpSHA, err := f.HeadSHA()` and then calls `appendWarpSHATrailer(message, warpSHA)`. No caller can choose the value. Card 12 gives up only when the tip is "not held locally", so the owner's HEAD is whatever its task worktree is at, and nothing guarantees that equals the previous transition's tip.
- **Concrete case:** the owner's task code branch sits behind the previous transition's tip. The give-up commit then names that older HEAD. Card 7's `resetUnnamedTip` reads it as "the tip the run's last transition names", sees origin's code tip as neither that tip nor an ancestor of it, and resets origin's code branch back to the older HEAD. That drops the previous transition's own code commits.
- **Second case:** the owner's HEAD holds a local commit origin lacks. `OriginCodeTip` reads the give-up's tip as missing again, so the code-tip wait never ends.
- **Possible fixes:** have the give-up write `Warp-SHA` as the previous transition's `Warp-SHA` explicitly, which the trailer commit cannot do today. Or require and check that the owner's warp HEAD equals that tip before it commits.

**LOW — card 6, `lastTransitionCodeTip` paragraph, and card 8's trailer value**
Changed text: "`lastTransitionCodeTip` returns the `Warp-SHA` of the newest commit on ref that carries `TransitionTrailerKey`", while card 8 writes `Value: shedrun.ResolveRunID(location, runID)`.

- **Summary:** readers select by key only, so the run id the seams write is never matched. On a records branch more than one run commits to, another run's transition reads as this run's last transition.
- **Evidence:** batten's seam commits from the prime (`battenCommitStatusDeps`: "The spawned push runs from the prime"), so its `Run-Transition` commits land on the prime records branch, which every run shares (overview, `owed-is-a-failed-push-record`: "the prime's records branch").
- **Why only LOW:** the readers in cards 6, 7 and 12 read the slug's own records branch, so nothing breaks today. The value is dead weight unless the readers match on it as well.

F2 and F3: I found no case where the proposal is wrong.

Tool calls: none.
