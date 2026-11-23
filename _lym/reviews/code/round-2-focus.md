---
round: 2
focus:
  - "In `internal/landingshed/prreview.go`, `PRReview.wake` and `haltOnEndedSession`: settle whether the session-liveness check can halt the wait before a status that ends the review (merge, approval, continue flag) has been read, on a wake that read no status."
  - "In `internal/landingshed/prreview.go`, `haltOnEndedSession`: settle whether a failed `SessionLive` read inside the wait behaves like the wait's other periodic reads (warn once per episode, keep waiting) or ends the call and stops a live session."
  - "In `internal/landingshed/prreview_wait_test.go`, `TestPRReview_Wait`: settle whether the halt on a wake that read the status and left the review open, and a failed liveness read inside the wait, are each reached by a row that fails when that behavior is removed."
  - "In `internal/landingshed/prreview.go`, the `keepsSession` flag and the `rec.ReviewEnded` guard in `haltOnEndedSession`: settle whether a test row pins the flag value its name claims."
---

Focus targets the code paths the round-1 findings touched, restated as questions for a fresh read.
