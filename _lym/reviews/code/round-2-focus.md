---
round: 2
focus:
  - "internal/shuttleengine/run.go, Runner.start: when a resumed attempt fails retryably and the retry loop runs again, does the next attempt launch a fresh session, with no ResumeSessionID, no resume preamble, and the carried-instructions block restored?"
  - "internal/shuttleengine/attach.go, the superseding attach that removes a respawn-eligible running candidate: do the tests pin that a running record with a dead pane is recorded stopped with Interrupted true, and a terminal outcome keeps Interrupted false?"
  - "internal/shuttleengine/run.go, resumed start: do the tests pin that carried operator instructions stay out of a resumed prompt, are still recorded in run.json, and send the notifier nothing?"
  - "internal/shuttleengine/stop.go StopStrand and crashresume.go RetireInterruptedRuns: do the tests pin that a deliberate stop records Interrupted false, and that a failed write does not stop the scan and comes back joined?"
  - "contracts/orchrole/orch-role.md and _lym/landing/summary.md: do the stencil's re-call clause and the Merge danger blast radius match what the diff does in shuttle's run start, run.json, and loom's goto and arbiter relaunch?"
---

Targets the round after a CONTINUE: the gating design findings first, then the test and wording gaps beside them.
