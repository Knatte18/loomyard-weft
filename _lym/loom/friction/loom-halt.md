loom halted failed at Code-Burler-Fix

```
loop-exited: loop 4fe669c4cdf90c6b ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/orch-launch-refused-by-gate-key/.lym/shed/orch-launch-refused-by-gate-key/steps/loop.log; step Code-Burler-Fix was in flight under trace 8b41c852f3bb2ab4; loop exit: not recorded, since no process marked the loop's exit, and the machine has booted since the loop started, so a stop of the machine likely ended it; stopped shuttle runs: ly:orch-launch-refused-by-gate-key:code-burler-fix
status error: shedverbs: step Code-Burler-Fix interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step orch-launch-refused-by-gate-key --until-stop
```

producer: Code-Burler-Fix
state: failed
anomaly: interrupted-stop
history_entries: 9
trace_file: /home/knatte/Code/loomyard-LYMHUB/orch-launch-refused-by-gate-key/.lym/logs/trace-20261122T102546Z-8b41c852f3bb2ab4-3270150.log

history:
- Preflight / done / 2026-11-22T09:53:21Z
- Record-Write / done / 2026-11-22T09:53:26Z
- Drift-Baseline / skipped / 2026-11-22T09:53:32Z
- Drift-Await / skipped / 2026-11-22T09:53:37Z
- Drift-Expect / skipped / 2026-11-22T09:53:43Z
- Darn-Write / done / 2026-11-22T10:16:34Z
- Drift-Verify / skipped / 2026-11-22T10:16:42Z
- Code-Bouncer / stuck / 2026-11-22T10:17:06Z / routed to Code-Burler-Review
- Code-Burler-Review / stuck / 2026-11-22T10:25:40Z / routed to Code-Burler-Fix
