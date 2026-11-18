loom halted failed at Parent-Align

```
loop-exited: loop db155b8ed56221d4 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/review-result-checks/.lym/shed/review-result-checks/steps/loop.log; step Parent-Align was in flight under trace 98ec2cb915e2d2fa; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; stopped shuttle runs: ly:review-result-checks:fixer
status error: shedverbs: step Parent-Align interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step review-result-checks --until-stop
```

producer: Parent-Align
state: failed
anomaly: interrupted-stop
history_entries: 16
trace_file: /home/knatte/Code/loomyard-LYMHUB/review-result-checks/.lym/logs/trace-20261118T130532Z-98ec2cb915e2d2fa-804834.log

history:
- Preflight / done / 2026-11-18T11:07:11Z
- Motif / skipped / 2026-11-18T11:07:23Z
- Drift-Baseline / skipped / 2026-11-18T11:07:29Z
- Parts-Write / skipped / 2026-11-18T11:07:35Z
- Parts-Plan-Write / skipped / 2026-11-18T11:07:47Z
- Plan-Write / done / 2026-11-18T11:27:06Z
- Plan-Bouncer / stuck / 2026-11-18T11:27:36Z
- Plan-Burler-Review / stuck / 2026-11-18T11:36:44Z
- Plan-Burler-Fix / stuck / 2026-11-18T11:42:07Z
- Plan-Bouncer / stuck / 2026-11-18T11:42:40Z
- Plan-Burler-Review / stuck / 2026-11-18T11:47:34Z
- Plan-Burler-Fix / stuck / 2026-11-18T11:50:51Z
- Plan-Bouncer / done / 2026-11-18T11:51:20Z
- Drift-Await / skipped / 2026-11-18T11:51:27Z
- Drift-Expect / skipped / 2026-11-18T11:51:37Z
- Webster / done / 2026-11-18T13:05:24Z
