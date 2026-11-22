loom halted failed at Discussion-Burler-Review

```
loop-exited: loop 67e304af4297cdc3 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/orch-full-autonomy-mode/.lym/shed/orch-full-autonomy-mode/steps/loop.log; step Discussion-Burler-Review was in flight under trace 79cf69a398d955bf; loop exit: not recorded, since no process marked the loop's exit, and the machine has booted since the loop started, so a stop of the machine likely ended it; stopped shuttle runs: ly:orch-full-autonomy-mode:discussion-burler-review
status error: shedverbs: step Discussion-Burler-Review interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step orch-full-autonomy-mode --until-stop
```

producer: Discussion-Burler-Review
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/orch-full-autonomy-mode/.lym/logs/trace-20261122T102125Z-79cf69a398d955bf-3200559.log

history:
- Discussion-Write / done / 2026-11-22T10:04:12Z
- Discussion-Bouncer / stuck / 2026-11-22T10:04:40Z / routed to Discussion-Burler-Review
- Discussion-Burler-Review / stuck / 2026-11-22T10:13:04Z / routed to Discussion-Burler-Fix
- Discussion-Burler-Fix / stuck / 2026-11-22T10:20:46Z / routed to Discussion-Bouncer
- Discussion-Bouncer / stuck / 2026-11-22T10:21:19Z / routed to Discussion-Burler-Review
