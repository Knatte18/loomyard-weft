loom halted failed at Darn-Write

```
loop-exited: loop 3ae59064c58c60da ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/redundancy-confirmed-survey/.lym/shed/redundancy-confirmed-survey/steps/loop.log; step Darn-Write was in flight under trace bd27c81c25b9c19f; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; loop log: time=2026-11-17T07:02:59.833+01:00 level=WARN msg="shuttle: a run reads running but its waiting process is gone" trace=db47cc2ffe07cab1 runDir=/home/knatte/Code/loomyard-LYMHUB/redundancy-confirmed-survey/.lym/shuttle/0553fb54f6600b8848017c88b1960081 strand=ly:redundancy-confirmed-survey:darn pid=2301815; step stderr: time=2026-11-17T07:03:49.092+01:00 level=WARN msg="shuttle: session state disagrees" trace=bd27c81c25b9c19f strandGUID=dfaf5452999ab53a7678c86170c07b95 loop=held state=busy cause=background stands=loop
status error: shedverbs: step Darn-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step redundancy-confirmed-survey --until-stop
```

producer: Darn-Write
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/redundancy-confirmed-survey/.lym/logs/trace-20261117T060259Z-bd27c81c25b9c19f-7001.log

history:
- Preflight / done / 2026-11-16T14:05:58Z
- Record-Write / done / 2026-11-16T14:06:04Z
- Drift-Baseline / skipped / 2026-11-16T14:06:21Z
- Drift-Await / skipped / 2026-11-16T14:06:33Z
- Drift-Expect / skipped / 2026-11-16T14:06:39Z
