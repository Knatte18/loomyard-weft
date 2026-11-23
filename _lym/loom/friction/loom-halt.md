loom halted failed at Webster

```
loop-exited: loop 1cf48a875601e129 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/async-run-pusher/.lym/shed/async-run-pusher/steps/loop.log; step Drift-Await was in flight under trace c2a88bc9ff156446, and the status file now reads Webster; loop exit: not recorded, since no process marked the loop's exit, and the machine has booted since the loop started, so a stop of the machine likely ended it; stopped shuttle runs: ly:async-run-pusher:webster
status error: shedverbs: step Webster interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step async-run-pusher --until-stop
```

producer: Webster
state: failed
anomaly: interrupted-stop
history_entries: 18
trace_file: /home/knatte/Code/loomyard-LYMHUB/async-run-pusher/.lym/logs/trace-20261123T183602Z-c2a88bc9ff156446-2007620.log

history:
- Preflight / done / 2026-11-23T13:49:03Z
- Motif / skipped / 2026-11-23T13:49:22Z
- Drift-Baseline / skipped / 2026-11-23T13:49:29Z
- Parts-Write / skipped / 2026-11-23T13:49:41Z
- Parts-Plan-Write / skipped / 2026-11-23T13:50:00Z
- Plan-Write / done / 2026-11-23T14:18:28Z
- Plan-Bouncer / stuck / 2026-11-23T14:19:06Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-23T14:35:21Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-23T18:20:13Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-23T18:20:49Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-23T18:27:05Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-23T18:31:24Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-23T18:32:04Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-23T18:34:35Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-23T18:35:27Z / routed to Plan-Bouncer
- Plan-Bouncer / done / 2026-11-23T18:35:56Z
- Drift-Await / skipped / 2026-11-23T18:36:02Z
- Drift-Expect / skipped / 2026-11-23T18:36:07Z
