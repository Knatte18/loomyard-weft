---
round: 1
focus:
  - "internal/landingshed/prreviewask.go ReadPRReviewStanding: settle whether a satisfied answer stands at the head when a later ask at that head is answered with findings, cleared, or of another tag; the loop's `continue` on Cleared and its `break` after the first answered ask decide each case."
  - "Same function: settle whether the Awaiting path and the final fallthrough still give the old results when no answered ask at head exists, and whether a satisfied answer that stands wrongly masks an ask-again the parent asked for."
  - "Scope against the amended board entry: items 1 and 2 stand as committed and delivered, item 3 is dropped. Settle whether the diff lands item 2, a parent verb to record a satisfied answer without a delivered ask and without a --head flag, and whether anything in the diff goes beyond items 1 and 2."
  - "The stencil line in landing-block-pr-review-autonomous.md and the doc.go addition: settle whether each claim matches the code's behavior and whether the stencil's neighbouring rules on await outcomes stay consistent with it."
  - "prreviewask_test.go: settle whether the new cases cover the later-ask states (awaiting, timeout, no-parent, undelivered) and the findings-after-satisfied case with independent literal expectations, without duplicating existing cases."
---

Seed focus from the committed diff and the decision record's Goal plus its one added entry.
