# Describe seat waits out the full timeout after an operator interrupt, then halts as hard failure

`describe-held-turn-waits-timeout` · Note · bug, landing, shuttle

Imported from [issue #241](https://github.com/Knatte18/loomyard/issues/241).

The Describe producer (seats engine) halted the run as `failed` with `shuttle run outcome timeout`, after its agent session sat held for about 28 minutes with no output.

Sequence from the trace:

```
22:15:05 shuttle: run started role=describe
22:16:51 shuttle: session state state=idle-stalled cause=interrupt
22:16:53 shuttle: session state state=busy cause=turn
22:16:57 shuttle: turn end held, output files missing
22:16:57 shuttle: held turn end of a turn the operator started; no hold notice
22:16:57 shuttle: session state state=idle-stalled cause=no-output loop=held
22:45:06 shuttle: run finished outcome=timeout cleanedUp=false
22:45:29 shed: step refused kind=producer transient="" error="shedadapters: Describe (seats): shuttle run outcome timeout"
```

The halt note reads `loom halted failed at Describe`, `anomaly: producer-hard-failure`, 12 history entries.

Why this is abnormal:
- The operator interrupted the agent once and typed into the pane, which made the turn "operator-started".
  lym then held the turn end without a hold notice and waited silently until the run timeout.
- The session was `idle-stalled` for the whole wait, and nothing re-prompted the agent for its missing output files or surfaced the state to the operator.
- The timeout was then recorded as a hard failure with `transient=""`, although the cause was an interrupted session and not a producer defect.
- `cleanedUp=false` on the finished run, and the strand was stopped with `live=true` afterwards.

Suggested fix: when a turn end is held for missing output files after an operator-started turn, send the agent a reminder or hand the pane back to the operator with a visible notice instead of waiting out the whole timeout.
Classify a timeout that follows an operator interrupt as resumable, not as `producer-hard-failure`.

Provenance: task `transient-tripout-overpull`, from `loom-halt.md`.
