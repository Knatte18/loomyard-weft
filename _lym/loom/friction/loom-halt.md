loom halted failed at Beam-Await

```
loop-exited: loop 0146016148faa753 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit/.lym/shed/adopt-build-resplit/steps/loop.log; step Beam-Await was in flight under trace 11e470e559a9a455; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine
status error: shedverbs: step Beam-Await interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step adopt-build-resplit --until-stop
```

producer: Beam-Await
state: failed
anomaly: interrupted-stop
history_entries: 3
trace_file: /home/knatte/Code/loomyard-LYMHUB/adopt-build-resplit/.lym/logs/trace-20261116T120129Z-11e470e559a9a455-2563010.log

history:
- Preflight / done / 2026-11-16T12:00:53Z
- Record-Write / done / 2026-11-16T12:00:58Z
- Beam-Up / done / 2026-11-16T12:01:22Z
