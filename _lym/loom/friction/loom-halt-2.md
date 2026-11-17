loom halted failed at Beam-Await

```
loop-exited: loop 9de9163a188192f8 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit/.lym/shed/adopt-build-resplit/steps/loop.log; step Beam-Await was in flight under trace 50f4be0fc0c5fd6f; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; loop log: time=2026-11-17T07:03:16.390+01:00 level=WARN msg="shuttle: a run reads running but its waiting process is gone" trace=9fd91b579875008d runDir=/home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit/.lym/shuttle/58a0b5dcdd6f968fb9ee520a9f6b9696 strand=ly:adopt-build-resplit:orch pid=2555061
status error: shedverbs: step Beam-Await interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step adopt-build-resplit --until-stop
```

producer: Beam-Await
state: failed
anomaly: interrupted-stop
history_entries: 3
trace_file: /home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit/.lym/logs/trace-20261117T060316Z-50f4be0fc0c5fd6f-11358.log

history:
- Preflight / done / 2026-11-16T12:00:53Z
- Record-Write / done / 2026-11-16T12:00:58Z
- Beam-Up / done / 2026-11-16T12:01:22Z
