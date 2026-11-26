loom halted failed at Darn-Write

```
loop-exited: loop d197ea3f57f043ad ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/sandbox-without-new-kernel/.lym/shed/sandbox-without-new-kernel/steps/loop.log; step Drift-Baseline was in flight under trace 5ac70403495e6883, and the status file now reads Darn-Write; loop exit: not recorded, since no process marked the loop's exit, and the machine has booted since the loop started, so a stop of the machine likely ended it; step stderr: time=2026-11-23T20:19:20.767+01:00 level=WARN msg="loomshed: verify gate found comment line findings" trace=5ac70403495e6883 gate="Darn-Write gate" attempt=1 count=15; stopped shuttle runs: ly:sandbox-without-new-kernel:gate-fixer; left runs whose output files all exist: ly:sandbox-without-new-kernel:darn
status error: shedverbs: step Darn-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step sandbox-without-new-kernel --until-stop
```

producer: Darn-Write
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/sandbox-without-new-kernel/.lym/logs/trace-20261123T190045Z-5ac70403495e6883-3815363.log

history:
- Preflight / done / 2026-11-23T19:00:32Z
- Record-Write / done / 2026-11-23T19:00:39Z
- Drift-Baseline / skipped / 2026-11-23T19:00:46Z
- Drift-Await / skipped / 2026-11-23T19:00:51Z
- Drift-Expect / skipped / 2026-11-23T19:00:57Z
