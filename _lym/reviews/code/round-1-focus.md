---
round: 1
focus:
  - "internal/shuttleengine/crashresume.go, run.go, carryover.go: does a re-called row resume the recorded session only when its run ended without finishing (crash, reboot, killed pane), for every role and engine?"
  - "internal/shuttleengine/crashresume.go: does a run that finished, was stopped on purpose, or has a missing transcript get a fresh agent rather than a resume?"
  - "internal/loomcli/gotoresume.go and cli.go: does an admitted goto set aside interrupted runs so a deliberate restart spawns a fresh agent, and do new review rounds and re-calls after an escalation answer also start fresh?"
  - "internal/shuttleengine/crashresume.go:92, setAside: a mutation survivor negates its condition; which test could pin what the condition decides?"
  - "internal/shuttleengine/attach.go:240 and run.go:651: survivors of a negated comparison and a deleted clause; does a test pin the behavior the diff changes there?"
  - "internal/shuttleengine/stop.go, stepstop.go, stop_test.go: do the stop changes keep their earlier behavior, including a stop that cannot be recorded removing nothing?"
  - "Existing resume paths (ResumeSessionID users such as accountrelaunch.go): does the new carryover change alter their behavior?"
---

The record's `## Goal` holds the board entry: resume the recorded session on a re-call after an interrupted run, with a fresh agent kept for deliberate restarts.
