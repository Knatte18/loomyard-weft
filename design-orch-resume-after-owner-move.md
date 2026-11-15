# The orch comes back as a new session, not a continuation, after the hub moves back to this machine

`orch-resume-after-owner-move` · Note · bug, orch, shuttle

## Seen (operator, 2026-11-15)

The operator worked on OSL-1046 while Neve hibernated (2026-11-12 21:44 to 2026-11-15 12:20).
The hub's orch watcher on Neve had exited with `owned by machine osl46` (`.lym/orch/watch.log`).
After the hub came back to Neve, the prime orch came up at 12:27 as a new Claude session (`7da34817…`).
That session got a fresh prompt that pointed at the 2026-11-12 handoff,
followed by the previous session's typed instructions carried over (`internal/shuttleengine/carryover.go`).
The previous orch session (`e7be957b…`, a 10 MB transcript on Neve) was not resumed.
The operator wants the orch to come back as a continuation of the previous orch session, never as a new one.

## Idea

When the prime or a beam orch comes up again after an ownership move or a restart, resume its last session from the transcript on this machine.
Fall back to a fresh launch with the carry-over only when no transcript is present here, and say so in the launch line.
First find which path chose the fresh launch: the watcher's exit on the ownership move, the adopt, or the orch launch itself.
