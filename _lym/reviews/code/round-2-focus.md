---
round: 2
focus:
  - "In internal/landingshed/pullrequestmode.go (resolveWithParentRule) and internal/landingshed/finalize_pr.go (Finalize's rule read before dropRecordedPullRequest): when a pull request record exists and the parent-rule read fails with a non-transient, non-token error, does Finalize halt rather than close the pull request and land by a direct push? Is the soft Warn path kept only for a run with no record?"
  - "In internal/landingshed/prreview.go (judge, StateOpen case, and openPullRequest): for an open pull request whose record carries ReviewEnded, on a clean and on a dirty tree, does the PR-Review wait end Done or Stuck with the dirty paths named, as the StateClosed case does, instead of polling forever?"
  - "In internal/landingshed/prreview_test.go, prreview_wait_test.go and finalize_pr_test.go: does a test row pin each behavior the diff adds in resolveWithParentRule, namely the token-failure soft-fail, the cancelled-context error, and the skips for an empty task branch and an unusable origin?"
---

Focus targets the behaviors the round 1 review found open; the rest of the range stays as settled.
