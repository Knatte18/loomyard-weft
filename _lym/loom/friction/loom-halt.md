loom halted failed at Sampler-Session

```
loop-exited: loop a69eaa618b2e417d ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/readme-declutter/.lym/shed/readme-declutter/steps/loop.log; step Drift-Baseline was in flight under trace 27b795be71720842, and the status file now reads Sampler-Session; loop exit: not recorded, since no process marked the loop's exit, and the machine has booted since the loop started, so a stop of the machine likely ended it; stopped shuttle runs: ly:readme-declutter:sampler
status error: shedverbs: step Sampler-Session interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step readme-declutter --until-stop
```

producer: Sampler-Session
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/readme-declutter/.lym/logs/trace-20261122T095801Z-27b795be71720842-1954480.log

history:
- Preflight / done / 2026-11-22T09:57:51Z
- Record-Write / done / 2026-11-22T09:57:56Z
- Drift-Baseline / skipped / 2026-11-22T09:58:01Z
- Drift-Await / skipped / 2026-11-22T09:58:09Z
- Drift-Expect / skipped / 2026-11-22T09:58:14Z
