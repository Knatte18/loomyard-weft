loom halted failed at Darn-Write

```
loop-exited: loop 91ba8f9c30a4de7e ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/rc-click-wrong-client-session/.lym/shed/rc-click-wrong-client-session/steps/loop.log; step Drift-Baseline was in flight under trace f3860128d6dda813, and the status file now reads Darn-Write; loop exit: not recorded, since no process marked the loop's exit, and the machine has booted since the loop started, so a stop of the machine likely ended it; stopped shuttle runs: ly:rc-click-wrong-client-session:darn
status error: shedverbs: step Darn-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step rc-click-wrong-client-session --until-stop
```

producer: Darn-Write
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/rc-click-wrong-client-session/.lym/logs/trace-20261123T191856Z-f3860128d6dda813-377453.log

history:
- Preflight / done / 2026-11-23T19:18:42Z
- Record-Write / done / 2026-11-23T19:18:50Z
- Drift-Baseline / skipped / 2026-11-23T19:18:56Z
- Drift-Await / skipped / 2026-11-23T19:19:02Z
- Drift-Expect / skipped / 2026-11-23T19:19:09Z
