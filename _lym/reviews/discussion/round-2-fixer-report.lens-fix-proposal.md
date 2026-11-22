```yaml
lens: fix-proposal
outcome: reported
```

The F23, F24 and F25 proposals each break in a concrete case. I found no case that breaks the F22, F26, F27 or F28 proposals.

Proposals checked: F22 (option A, the PR-Review asks kept in the run's ask record), F23, F24, F25, F26, F27, F28.

1. **MEDIUM**: D4's closed-pull-request bullet ("Finalize never reopens the pull request (`internal/landingshed/finalize_pr.go`)") and the matching acceptance bullet ("a pull request closed on GitHub halts PR-Review, `lym loom pr-continue` is refused on it, and Finalize never reopens it").
   - Summary: the proposal covers only a pull request that is closed while PR-Review waits. If GitHub closes it after an autonomous review has already ended, Finalize still reopens it and merges it.
   - The case: under autonomy the parent answers satisfied, and `pr-continue` sets the continue flag. The row's `judge` then calls `endReview`, which records `ReviewEnded` true (`internal/landingshed/prreview.go`, `endReview`), and PR-Review is done. Before Finalize runs, someone closes the pull request on GitHub.
   - Evidence: `internal/landingshed/finalize_pr.go:358-368`. On `StateClosed`, Finalize halts only when `!rec.ReviewEnded`. Otherwise it calls `api.Reopen` and merges, so a close made on GitHub is overridden by no one's word. That is the defect F24 named, and D4 still calls a closed pull request to reopen a class 4 severe halt.

2. **MEDIUM**: D5's class 5 counting ("Each entry at the row counts as one halt plus its `repeats` …"), its Bound ("counted from the status history with each entry's `repeats`"), and the Review D5 acceptance bullet.
   - Summary: `repeats` also counts identical budget-exempt Stucks that the driver steps again with no halt or resume between them. A row that bounced before its first real halt can therefore reach the class 5 count on that first halt.
   - Evidence: `internal/shedengine/step_fold_test.go` (`TestStep_NoVerdictWriteCarriesActivityLastAfterFold`). Two plain `mustStep` calls on a scripted stuck shed fold into one entry with `Repeats` 1, carrying `foldedBounceWording`, before any pause or resume. A further step after the pause makes `Repeats` 2.
   - `appendOrFold` and `foldsInto` (`internal/shedengine/run.go:706-740`) fold on Producer, Output, RoutedTo and GateAttempts alone, never on whether the run halted. So an entry plus its repeats is not a count of the halts the orch settled.

3. **MEDIUM**: D4's rewritten Bound ("only a recorded satisfied answer at the head the row judges ends the review: `pr-continue` checks it before it sets the continue flag, and the row's `judge` checks it again before it ends on the flag").
   - Summary: the F23 proposal gates only the continue-flag ending. The row's `judge` also ends the review, with no parent answer, on a GitHub approval of the gated head and on a merge made on GitHub, so the "only" claim does not hold.
   - Evidence: `internal/landingshed/prreview.go`, `judge`. In `StateOpen`, `status.Approved && !status.ChangesRequested && status.HeadSHA == rec.GatedHead && status.HeadSHA == head && clean` returns `done()`. In `StateMerged`, a clean tree returns `done()`. Neither path reads an ask.

No further cases found against F22, F26, F27 or F28.

Tool calls:
- Read `internal/landingshed/finalize_pr.go:335-372` and `internal/shedengine/step_fold_test.go:85-120`. My context held only grep hits for Finalize's reopen path and none of the fold test's bodies.
