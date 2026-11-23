# Verify-budget notice names no run

`verify-budget-notice-no-slug` · Task · bug, loom · draft

## Seen

The orch got creel mail #2450 from `ly:LOOM` (2026-11-23):
"Darn-Write gate: the verify gate has run past its 30m0s budget and is at attempt 3; it goes on until its attempts are spent; no verify run has failed yet".
Neither the sender nor the text names the run, and three darn runs were in Darn-Write at the time,
so the orch could only guess which run it meant.
The line is built by `verifyBudgetNotice` in `internal/loomshed/verifybudget.go`, which takes the gate site but no slug.

## Idea

Put the run's slug in the line, as other parent notices do, so the parent knows which run it is about without a status read.
