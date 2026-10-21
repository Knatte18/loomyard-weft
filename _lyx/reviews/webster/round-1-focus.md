---
round: 1
focus:
  - "Cards 2, 4, 5, 6, 11, 12, 13: check each new seam (`Subject`, `GotoGuard`, `RejectReview`, `BudgetWayForward`) is nil or empty by default and leaves today's path unchanged; settle whether any default-path behavior differs."
  - "Cards 10, 11, 12, 14: check the settle record, its patch and the generation record are written through `internal/state` in the Bouncer's run directory with locks beside them, and survive an archive; settle whether any write bypasses `internal/state` or is non-atomic."
  - "Cards 2, 3, 6, 16: check the budget halt's paused reason is recorded and read consistently, that only the parent's resume lifts a bounce-budget-exhausted halt, and that a goto never grants a fresh budget; settle whether any path contradicts this."
  - "Card 11 and 12: check an unchanged subject re-settles with no review, and a changed subject starts an amendment generation scoped to the change under `review_amend_max_bounces`; settle edge cases around the patch-id comparison and merge-in-only history (card 8)."
  - "Card 18 and the `PATTERN-friction-capture` change: check the diff lands the declared PATTERN text, and that a stop starts no friction session while a halt still writes its note."
  - "Card 19: check a review round with the wrong fork count is retried and the chair's ending is named; settle whether the retry is bounded."
  - "Test economy across the diff: new tests that duplicate an existing test, and added behavior with no coverage."
---

Initial focus drawn from the plan's Card Index and Shared Decisions; no round has been judged yet.
