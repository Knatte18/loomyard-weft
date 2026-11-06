## Changes

- F2: `internal/beamcli/mergeparent.go` — restored the space in `Filed by `lym beam merge-parent X``. Commit `ec4b5422`. No test added: the finding is a text regression, not a coverage gap.
- F3: `internal/boardengine/task.go` — the `normalizeTask` doc sentence now says a PullRequest other than empty, on, off and `PullRequestMergeIn` is refused. Commit `f7984698`.

F1 is `decision`-class and went to the parent; it is left out of this report.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

- F4, ground `premise false`: the review says nothing asserts the merge-in row reads no GitHub rule, so deleting `decision.TrueMerge` from the early return in `resolveWithParentRule` (`internal/landingshed/pullrequestmode.go:159`) leaves every test green.
  The row `AMergeInEntryLandsByATrueMergeWithNoPullRequestAndNoRuleReadUnderARequiredParent` (`internal/landingshed/finalize_pr_test.go:229`) sets no `wantCalls`, and the test ends with `cmp.Diff(tt.wantCalls, api.calls)` (`finalize_pr_test.go:555`).
  `fakePullRequestAPI.RequiresPullRequest` records every call through `f.next` (`internal/landingshed/prreview_test.go:114-116`).
  A rule read for the merge-in entry would therefore add `RequiresPullRequest main` to `api.calls` and fail the row's diff against the empty want.
  I read this from the code and did not run the mutation.

## Deferred

None.
