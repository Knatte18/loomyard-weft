loom halted failed at Code-Burler-Fix

```
loop-exited: loop 19a2b3b1051ba475 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/orch-launch-refused-by-gate-key/.lym/shed/orch-launch-refused-by-gate-key/steps/loop.log; loop exit: lym shed: loop 19a2b3b1051ba475 received signal terminated and ends without an envelope | lym shed: loop 19a2b3b1051ba475 process 11009 exited: signal: terminated; loop log: time=2026-11-22T11:44:23.170+01:00 level=WARN msg="shed: loop received a terminating signal and ends without an envelope" trace=8739c030ac31996d loop_id=19a2b3b1051ba475 signal=terminated
status error: shedverbs: step Code-Burler-Fix interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step orch-launch-refused-by-gate-key --until-stop
```

producer: Code-Burler-Fix
state: failed
anomaly: interrupted-stop
history_entries: 9
trace_file: /home/knatte/Code/loomyard-LYMHUB/orch-launch-refused-by-gate-key/.lym/shed/orch-launch-refused-by-gate-key/steps/loop.log

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
