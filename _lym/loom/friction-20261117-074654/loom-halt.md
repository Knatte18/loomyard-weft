loom halted failed at Webster

```
loop-exited: loop 64f4aa52d4dd8e8d ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/transition-push-takeover/.lym/shed/transition-push-takeover/steps/loop.log; step Drift-Await was in flight under trace ac89001134dd5e37, and the status file now reads Webster; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; step stderr: time=2026-11-16T15:09:36.665+01:00 level=WARN msg="shuttle: session state disagrees" trace=ac89001134dd5e37 strandGUID=13630fce5704e03fd908560054a969a3 loop=held state=busy cause=background stands=loop
status error: shedverbs: step Webster interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step transition-push-takeover --until-stop
```

producer: Webster
state: failed
anomaly: interrupted-stop
history_entries: 19
trace_file: /home/knatte/Code/loomyard-LYMHUB/transition-push-takeover/.lym/logs/trace-20261116T135452Z-ac89001134dd5e37-2193774.log

history:
- Preflight / done / 2026-11-16T12:02:33Z
- Motif / skipped / 2026-11-16T12:02:48Z
- Drift-Baseline / skipped / 2026-11-16T12:02:50Z
- Parts-Write / skipped / 2026-11-16T12:02:53Z
- Parts-Plan-Write / skipped / 2026-11-16T12:02:56Z
- Plan-Write / done / 2026-11-16T13:05:26Z
- Plan-Bouncer / stuck / 2026-11-16T13:06:05Z
- Plan-Burler-Review / stuck / 2026-11-16T13:18:06Z
- Plan-Burler-Fix / stuck / 2026-11-16T13:26:17Z
- Plan-Bouncer / stuck / 2026-11-16T13:26:51Z
- Plan-Burler-Review / stuck / 2026-11-16T13:34:49Z
- Plan-Burler-Fix / stuck / 2026-11-16T13:41:30Z
- Plan-Bouncer / stuck / 2026-11-16T13:42:08Z
- Plan-Burler-Review / stuck / 2026-11-16T13:47:52Z
- Plan-Burler-Fix / stuck / 2026-11-16T13:52:52Z
- Plan-Bouncer / awaiting / 2026-11-16T13:53:37Z
- Plan-Bouncer / done / 2026-11-16T13:54:43Z
- Drift-Await / skipped / 2026-11-16T13:54:53Z
- Drift-Expect / skipped / 2026-11-16T13:54:55Z
