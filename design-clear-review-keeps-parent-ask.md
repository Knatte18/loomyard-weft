# ClearReviewSession may keep the PR-Review parent ask after goto

`clear-review-keeps-parent-ask` · Note · bug, landing, loom

## Seen

`orch-autonomy-mode` landed as `30ed1ee58` with finding `clear-review-session-keeps-parent-review-asked` (design, MEDIUM) open:
`ClearReviewSession` in `internal/pullrequest` may leave the PR-Review parent ask state set after `lym loom goto`.

## Idea

Clear the ask state with the session, with a test.
