---
round: 2
focus:
  - "internal/shuttleparity/seats.go, the Holders of the DescribeTable, FixerTable and FixReviewSeatTable entries: does the glyph internal/landingshed#soloSession.table appear once, as a named constant, with no repeated literal?"
  - "internal/landingshed/finalize.go, Finalize.landUnderLock: parentHoldsWork feeds ownerChecks, whose run-scope reconcile can fast-forward the task branch; do landCheck and syncParent see an answer read after that reconcile, and does a failed read halt with the owner-halt reason?"
  - "internal/landingshed/finalize_pr.go, Deps.landingSummary, with its callers Finalize.Call and PRReview.pullRequestText: does each caller still return the error text it returned before the simplification, for a missing or malformed change description?"
  - "internal/landingshed/parentalign.go, ParentAlign.readAttempts and readRowAttempts: is the row entry read, and its error returned, before any scratch attempt record is moved, and does Finalize keep its own order?"
  - "internal/landingshed/prreview.go, checkWiring, and the wiring test in internal/landingshed/prreview_test.go: does a test leave each name in optionalPullRequestSeams unset and show Call raises no wiring error for it?"
  - "_lym/landing/summary.md: do the stated comment-line and code-line figures match a fresh run of go run ./tools/codestats -root internal/landingshed at HEAD?"
---

Each entry names a site the round-1 fixes touch and the question a fresh reader settles there.
