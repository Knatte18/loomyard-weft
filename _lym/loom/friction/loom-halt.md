loom halted failed at Code-Burler-Review

```
loop-exited: loop 9e363a89c832fd83 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/lym-retro/.lym/shed/lym-retro/steps/loop.log; step Code-Burler-Review was in flight under trace 1e7c2286ae2042dd; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine
status error: shedverbs: step Code-Burler-Review interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step lym-retro --until-stop
```

producer: Code-Burler-Review
state: failed
anomaly: interrupted-stop
history_entries: 11
trace_file: /home/knatte/Code/loomyard-LYMHUB/lym-retro/.lym/logs/trace-20261116T144631Z-1e7c2286ae2042dd-4130856.log

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
