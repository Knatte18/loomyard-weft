---
round: 2
focus:
  - "internal/shuttleengine/wait.go pollEventsTick and expiredTurnEnd: after a gated shell-only waiting turn end, when a later tick reads a turn start past the waiting turn end, is the waiting list cleared before the bound can return OutcomeDone, so no Done is counted while the agent is mid-turn? Check the engines without the SessionCycler idle reading too."
  - "internal/shuttleengine/wait.go queueShellNudge and sendShellNudge: when the send is dropped under run.covered (pause) or fails, are the shells left unnamed so a later waiting turn end over them queues the nudge again? Check that nudgedShells is written only after a successful send."
  - "internal/shuttleengine/waiting_test.go: does a row reach the turnStartAfter guard that skips queueing the nudge, the pause drop, and the failed send with no retry and Done still reached at the bound? Name the behavior any of these leaves without a failing test if removed."
  - "Doc comments at internal/shuttleengine/waitmark.go shellWaitActive, internal/shuttleengine/spec.go AwaitedShellPrefixes and internal/websterengine/runlevel.go masterAwaitedShellPrefixes: does each describe the gated shell-only waiting turn end as the code now behaves (held up to background_shell_wait_min with every output file present, judged as a plain turn end only with an output file missing)?"
---

The focus follows the open round 1 findings and asks only where to look.
