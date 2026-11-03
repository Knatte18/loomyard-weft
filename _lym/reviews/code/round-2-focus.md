---
round: 2
focus:
  - "internal/shuttleengine/carryover.go `unfinished`: settle whether a recorded api-error outcome, which `attach.go` treats as terminal, is carried as unfinished the way stopped, died and timeout are, so a later settler attempt under the same role key keeps the earlier attempt's carried and operator instructions."
  - "internal/sheddriver/tackler.go and internal/sheddriver/tackler_test.go: settle whether a test fails if the tackler's hand-back question stops naming the API error through `HandBackQuestionCause`."
  - "internal/shuttleengine/apierrorretry.go `retryAPIError` and `TestWait_APIErrorRetry` in apierrorretry_test.go: settle whether a test fails if a later turn start, turn end or session start no longer clears a pending API error, and whether the pause-covered, failed-send and earlier-errors-at-attach branches are exercised."
  - "internal/sheddriver/agentshuttle.go and every `shuttle.Spec` construction carrying a Timeout for an autonomous role: settle whether each role's handling of the api-error outcome matches the Goal's request that a StopFailure of a role run under a deadline is re-sent a bounded number of times, and whether each opted-in role maps the outcome in its own outcome switch."
---

Focus targets the three gating areas of the round and the scope question the diff leaves open.
