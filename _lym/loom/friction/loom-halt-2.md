loom halted failed at Code-Burler-Review

```
loop-exited: loop 2f1712570c5ae9cc ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/lym-retro/.lym/shed/lym-retro/steps/loop.log; step Code-Burler-Review was in flight under trace f4c1b27574e2a2f0; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; loop log: time=2026-11-17T07:02:58.778+01:00 level=WARN msg="shuttle: a run reads running but its waiting process is gone" trace=6b7a29337d299c93 runDir=/home/knatte/Code/loomyard-LYMHUB/lym-retro/.lym/shuttle/9a3d6355a08b5c3bdb8d37ede955ef1d strand=ly:lym-retro:code-burler-review pid=4130856
status error: shedverbs: step Code-Burler-Review interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step lym-retro --until-stop
```

producer: Code-Burler-Review
state: failed
anomaly: interrupted-stop
history_entries: 11
trace_file: /home/knatte/Code/loomyard-LYMHUB/lym-retro/.lym/logs/trace-20261117T060258Z-f4c1b27574e2a2f0-6392.log

history:
- Preflight / done / 2026-11-16T14:07:36Z
- Record-Write / done / 2026-11-16T14:07:39Z
- Drift-Baseline / skipped / 2026-11-16T14:07:43Z
- Drift-Await / skipped / 2026-11-16T14:07:48Z
- Drift-Expect / skipped / 2026-11-16T14:08:01Z
- Darn-Write / done / 2026-11-16T14:28:38Z
- Drift-Verify / skipped / 2026-11-16T14:28:43Z
- Code-Bouncer / stuck / 2026-11-16T14:29:15Z
- Code-Burler-Review / stuck / 2026-11-16T14:35:59Z
- Code-Burler-Fix / stuck / 2026-11-16T14:45:49Z
- Code-Bouncer / stuck / 2026-11-16T14:46:21Z
