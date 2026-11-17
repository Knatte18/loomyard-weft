loom halted failed at Webster

```
loop-exited: loop 5efa620d49b63a29 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/weaver-implementer/.lym/shed/weaver-implementer/steps/loop.log; step Webster was in flight under trace 05a467d952d94e47; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; loop log: time=2026-11-17T07:03:47.102+01:00 level=WARN msg="shuttle: a run reads running but its waiting process is gone" trace=5344ea1edfc131e0 runDir=/home/knatte/Code/loomyard-LYMHUB/weaver-implementer/.lym/shuttle/074e2ecc3199cebc1c693dcad11697d3 strand=ly:weaver-implementer:webster pid=2406515 | time=2026-11-17T07:03:47.103+01:00 level=WARN msg="shuttle: a run reads running but its waiting process is gone" trace=5344ea1edfc131e0 runDir=/home/knatte/Code/loomyard-LYMHUB/weaver-implementer/.lym/shuttle/2c944c52ab98841c9e7e7f6915061931 strand=ly:weaver-implementer:gate-fixer pid=4051213
status error: shedverbs: step Webster interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step weaver-implementer --until-stop
```

producer: Webster
state: failed
anomaly: interrupted-stop
history_entries: 19
trace_file: /home/knatte/Code/loomyard-LYMHUB/weaver-implementer/.lym/logs/trace-20261117T060347Z-05a467d952d94e47-20639.log

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
