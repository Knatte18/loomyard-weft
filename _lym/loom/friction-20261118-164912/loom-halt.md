loom halted failed at Darn-Write

```
loop-exited: loop 68d42388c3cd23c3 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/redundant-check-fixes/.lym/shed/redundant-check-fixes/steps/loop.log; step Drift-Baseline was in flight under trace 46a685a3b8c0c443, and the status file now reads Darn-Write; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; stopped shuttle runs: ly:redundant-check-fixes:darn
status error: shedverbs: step Darn-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step redundant-check-fixes --until-stop
```

producer: Darn-Write
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/redundant-check-fixes/.lym/logs/trace-20261118T125351Z-46a685a3b8c0c443-4189069.log

history:
- Preflight / done / 2026-11-18T12:53:38Z
- Record-Write / done / 2026-11-18T12:53:44Z
- Drift-Baseline / skipped / 2026-11-18T12:53:51Z
- Drift-Await / skipped / 2026-11-18T12:53:58Z
- Drift-Expect / skipped / 2026-11-18T12:54:03Z
