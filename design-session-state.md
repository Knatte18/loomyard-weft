# One session-state model in shuttle that every module reads

`session-state` · Note · enhancement, shuttle

Decided in direction by the operator 2026-10-07: unify how lyx knows what an agent session is doing, owned by shuttle.

## Why

Every module guesses an agent's state its own way, and each guess has failed:

- Batten judges a run quiet from the child's status file, which changes only at a step's end; on 2026-10-07 an API error stalled all five runs and the quiet notices came 45 min later, long after the agents were resumed by hand, naming no cause.
- The orch watcher runs its own idle probe and reads turn ends, and its compact cycle missed its own compaction (`orch-compact-timeout`).
- Shuttle's turn reading misses Stops and waits on finished background shells (#411, #420; `shuttle-turns`).
- An injected text races the operator's typing between the idle probe and the Enter (see `creel`).
- Nothing notices a live session stopped by an API error, machine sleep or a dropped network; elapsed time and transcript growth cannot tell it apart from a long simulation or test run.

## Direction

Shuttle owns one session-state model per agent strand, and batten, the orch watcher, creel and reed read it instead of inferring their own:

- `busy`: a turn is running, however long; a long tool call or simulation is never mistaken for a stall.
- `idle-done`: the turn ended with its output.
- `idle-stalled`: the session is idle but its step is not done, or the turn ended on an API error or an interrupt; detected in seconds from state, not from a timeout.
- `asking`: the agent waits on an answer.
- `dead`: the pane or process is gone.

Each state change is an event with a timestamp, so a consumer reacts to the change and keeps a history of changes only, never ticks (see `orch-notice-channel`).
`idle-stalled` gets a bounded resume: shuttle tells the session to continue, with a retry cap and backoff, then halts the run visibly with the cause.

## Relations

`shuttle-turns` fixes turn reading and is a first piece of this; this note is the step after it lands.
`run-recovery` covers dead processes; `orch-notice-channel` and `orch-compact-timeout` are consumers that would read this state; `creel` stage 2 delivers on it.
