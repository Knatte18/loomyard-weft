loom halted failed at Darn-Write

```
loop-exited: loop c31eb37f26ca70ab ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/redundancy-confirmed-survey/.lym/shed/redundancy-confirmed-survey/steps/loop.log; step Drift-Baseline was in flight under trace 39b74f87c112c498, and the status file now reads Darn-Write; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; step stderr: time=2026-11-16T15:07:23.581+01:00 level=WARN msg="shuttle: session state disagrees" trace=39b74f87c112c498 strandGUID=8800ee55199860075130f2139e8fd19d loop=held state=busy cause=background stands=loop | time=2026-11-16T15:15:38.728+01:00 level=WARN msg="shuttle: session state disagrees" trace=39b74f87c112c498 strandGUID=8800ee55199860075130f2139e8fd19d loop=held state=busy cause=background stands=loop
status error: shedverbs: step Darn-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step redundancy-confirmed-survey --until-stop
```

producer: Darn-Write
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/redundancy-confirmed-survey/.lym/logs/trace-20261116T140621Z-39b74f87c112c498-2301815.log

history:
- Preflight / done / 2026-11-16T14:05:58Z
- Record-Write / done / 2026-11-16T14:06:04Z
- Drift-Baseline / skipped / 2026-11-16T14:06:21Z
- Drift-Await / skipped / 2026-11-16T14:06:33Z
- Drift-Expect / skipped / 2026-11-16T14:06:39Z
