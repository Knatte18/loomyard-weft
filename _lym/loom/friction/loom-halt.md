loom halted failed at Plan-Write

```
loop-exited: loop 249203bfa9b89e34 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/pwc-opus/.lym/shed/pwc-opus/steps/loop.log; step Plan-Write was in flight under trace b9e9ca2ee80bda3d; loop exit: lym shed: loop 249203bfa9b89e34 received signal terminated and ends without an envelope | lym shed: loop 249203bfa9b89e34 process 1417827 exited: signal: terminated; loop log: time=2026-11-12T18:25:45.586+01:00 level=WARN msg="shed: loop received a terminating signal and ends without an envelope" trace=857896af66313fa1 loop_id=249203bfa9b89e34 signal=terminated
status error: shedverbs: step Plan-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step pwc-opus --until-stop
```

producer: Plan-Write
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/pwc-opus/.lym/logs/trace-20261112T172237Z-b9e9ca2ee80bda3d-1417845.log

history:
- Preflight / done / 2026-11-12T17:22:04Z
- Motif / skipped / 2026-11-12T17:25:00Z
- Drift-Baseline / skipped / 2026-11-12T17:25:01Z
- Parts-Write / skipped / 2026-11-12T17:25:02Z
- Parts-Plan-Write / skipped / 2026-11-12T17:25:03Z
