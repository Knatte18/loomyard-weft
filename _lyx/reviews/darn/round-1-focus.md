---
round: 1
focus:
  - "In `gitrepo`'s `namesRef`: does the short-form match stay exact on ref boundaries, so a line naming `archive/x/abc` never matches a different ref that merely shares a prefix or suffix, and a full tag ref never matches a branch line of the same name?"
  - "In `PushWithRetry` and its callers: does the new delays parameter keep every plain push primitive on `PushBackoffDelays()` with the same attempt count and schedule as before, and is the 'one more attempt than delays' arithmetic right at the empty and single-delay edges?"
  - "In `archiveTagPushDelays` and `archiveWeftTip`: does the schedule match the summary's claim (2 s doubling to 32 s, six attempts), and does the archive push run outside any lock that other writers wait on?"
  - "Against the board entry: does the diff make both changes the operator decision asks for, a longer backoff and coverage of the rejection class actually seen, without licensing extra scope?"
  - "Test economy: do the new `TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine` rows, the `PushWithRetry` row and the integration test each cover distinct behavior, and is any added behavior uncovered?"
  - "Docs: do the `gitrepo` and `fabricengine` package docs state the full-ref matching and the archive schedule as the code now behaves, and does the summary claim anything the diff does not bear out?"
---

Seed focus drawn from the board entry (#804) and the writer's summary.
