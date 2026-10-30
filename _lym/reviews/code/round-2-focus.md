---
round: 2
focus:
  - "internal/landingshed/prreview.go `wake` and `judge`: settle whether a pull request merged on GitHub at a task head the row never gated reaches `judge`'s merged-with-other-head halt, or is gated and pushed first and recorded as gated, and whether `wake` gates or pushes onto an already merged or closed pull request."
  - "internal/landingshed/prreview.go `closeRecordedPullRequest` and `endSession`, and internal/landingshed/prsession.go `ensureSession`: settle whether any path leaves a started session live and unrecorded, or removes the pull-request record before the session recorded in it is stopped."
  - "internal/pairteardown/pullrequest.go gone-worktree Warn and internal/landingshed/prreview.go `holdWhileCovered` doc comment: settle whether each states what the code does, including how often the pause-record error is logged and whether the Warn names the task branch."
  - "internal/landingshed/prreview_wait_test.go `TestPRReview_Wait`: settle whether each branch of the wait guard at prreview.go around line 641 has a row, notably an open pull request with the continue flag set on a clean tree at a head other than the gated head."
---

Focus entries name where to look; they restate no verdict.
