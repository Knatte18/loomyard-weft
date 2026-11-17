loom halted failed at Webster

```
loop-exited: loop 845e5633b79a9402 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/transition-push-takeover/.lym/shed/transition-push-takeover/steps/loop.log; step Webster was in flight under trace 240b995f66bd7000; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; loop log: time=2026-11-17T07:03:53.980+01:00 level=WARN msg="shuttle: a run reads running but its waiting process is gone" trace=c3db509b888f8a65 runDir=/home/knatte/Code/loomyard-LYMHUB/transition-push-takeover/.lym/shuttle/58e91b6545e90f3c6814604d6d4479d1 strand=ly:transition-push-takeover:gate-fixer pid=4072324 | time=2026-11-17T07:03:54.001+01:00 level=WARN msg="shuttle: a run reads running but its waiting process is gone" trace=c3db509b888f8a65 runDir=/home/knatte/Code/loomyard-LYMHUB/transition-push-takeover/.lym/shuttle/f4774409449da2c8b0f6ac9282a5522b strand=ly:transition-push-takeover:webster pid=2193774
status error: shedverbs: step Webster interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step transition-push-takeover --until-stop
```

producer: Webster
state: failed
anomaly: interrupted-stop
history_entries: 19
trace_file: /home/knatte/Code/loomyard-LYMHUB/transition-push-takeover/.lym/logs/trace-20261117T060354Z-240b995f66bd7000-27183.log

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
