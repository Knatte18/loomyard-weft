loom halted failed at Webster

```
loop-exited: loop f051084a83c698fa ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/weaver-implementer/.lym/shed/weaver-implementer/steps/loop.log; step Drift-Await was in flight under trace a7d4a2065a87dfb6, and the status file now reads Webster; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine
status error: shedverbs: step Webster interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step weaver-implementer --until-stop
```

producer: Webster
state: failed
anomaly: interrupted-stop
history_entries: 19
trace_file: /home/knatte/Code/loomyard-LYMHUB/weaver-implementer/.lym/logs/trace-20261116T140956Z-a7d4a2065a87dfb6-2406515.log

history:
- Preflight / done / 2026-11-16T12:08:33Z
- Motif / skipped / 2026-11-16T12:08:38Z
- Drift-Baseline / skipped / 2026-11-16T12:08:52Z
- Parts-Write / skipped / 2026-11-16T12:08:54Z
- Parts-Plan-Write / skipped / 2026-11-16T12:09:00Z
- Plan-Write / done / 2026-11-16T13:22:24Z
- Plan-Bouncer / stuck / 2026-11-16T13:22:58Z
- Plan-Burler-Review / stuck / 2026-11-16T13:34:46Z
- Plan-Burler-Fix / stuck / 2026-11-16T13:43:35Z
- Plan-Bouncer / stuck / 2026-11-16T13:44:08Z
- Plan-Burler-Review / stuck / 2026-11-16T13:50:15Z
- Plan-Burler-Fix / stuck / 2026-11-16T13:55:36Z
- Plan-Bouncer / stuck / 2026-11-16T13:56:15Z
- Plan-Burler-Review / stuck / 2026-11-16T14:01:59Z
- Plan-Burler-Fix / stuck / 2026-11-16T14:08:14Z
- Plan-Bouncer / awaiting / 2026-11-16T14:08:56Z
- Plan-Bouncer / done / 2026-11-16T14:09:53Z
- Drift-Await / skipped / 2026-11-16T14:09:56Z
- Drift-Expect / skipped / 2026-11-16T14:10:03Z
