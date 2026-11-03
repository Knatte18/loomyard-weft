---
round: 1
focus:
  - "internal/shuttleengine/apierrorretry.go, run.go, wait.go, waitshadow.go: does a StopFailure event end the turn, re-send after a back-off a bounded number of times, then end with api-error; settle whether the retry counter, back-off and deadline interplay can loop, double-send or miss an event."
  - "internal/shedarbiter and internal/sheddriver (tackler.go, agentshuttle.go, both prompts): does a settler whose session ended api-error hand the stop on to the parent with the error named, so the parent orch is told the stop and the failed settler; settle whether any role under a deadline is left silent."
  - "Scope against the Goal's Expected section: which settler roles besides arbiter and tackler run under a deadline, and does the diff cover them or leave a path unhandled."
  - "Tests in apierrorretry_test.go and the changed fakes: does each new behavior (retry, bound reached, hand-on) have coverage, and does any new test duplicate an existing one."
  - "Doc and comment conventions in the changed doc.go files and new doc comments, plus the commit messages of the range."
---

Seed focus drawn from the board entry's Expected section and the diff's file list.
