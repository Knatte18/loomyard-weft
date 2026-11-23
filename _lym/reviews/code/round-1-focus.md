---
round: 1
focus:
  - "internal/landingshed/prreview.go `wake`, the `else if end := p.haltOnEndedSession` branch near line 735 (mutation survivor): which wake paths reach it, and does any test fail when its condition is negated?"
  - "`wake` after the `!polled` early return and after the stay-open and resume branches: is `haltOnEndedSession` checked on every wake that leaves the review open, and never on a wake that ends or reroutes the review?"
  - "`haltOnEndedSession` and `keepsSession`: which `openPullRequest` paths pass true or false, and does each match the session the call actually started or reused (closed, merged, open, already-ended record)?"
  - "`SessionLive(rec.StrandGUID)`: does a transient read failure, an empty `StrandGUID` or a session not yet registered right after `ensureSession` wrongly halt or error the wait?"
  - "The halt's way-forward text against the issue: does `lym loom resume` re-enter the row and resume the recorded session as the message claims?"
  - "Test `TestPRReview_Wait` and `TestPRReview_WaitWithNoLastSetTextTakesTheDescriptionAsTheBaselineAndEditsNothing`: do the changed tests still check what their names state, and is the dead-session halt covered with an independent expected value?"
---

Seed focus for the dead-session halt in the PR-Review wait, from the diff, the issue text under `## Goal`, and the mutation survivor.
