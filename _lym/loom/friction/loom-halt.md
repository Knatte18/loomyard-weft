loom halted failed at Plan-Write

```
loop-exited: loop 9506ff80cbfdc53a ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/pwc-fable/.lym/shed/pwc-fable/steps/loop.log; step Plan-Write was in flight under trace 162a1ca6f5fffcb5; loop exit: lym shed: loop 9506ff80cbfdc53a received signal terminated and ends without an envelope | lym shed: loop 9506ff80cbfdc53a process 1439035 exited: signal: terminated; loop log: time=2026-11-12T18:25:45.878+01:00 level=WARN msg="shed: loop received a terminating signal and ends without an envelope" trace=cc4799ce926f8005 loop_id=9506ff80cbfdc53a signal=terminated
status error: shedverbs: step Plan-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step pwc-fable --until-stop
```

producer: Plan-Write
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/pwc-fable/.lym/logs/trace-20261112T172537Z-162a1ca6f5fffcb5-1439079.log

history:
- Preflight / done / 2026-11-12T17:22:08Z
- Motif / skipped / 2026-11-12T17:25:00Z
- Drift-Baseline / skipped / 2026-11-12T17:25:01Z
- Parts-Write / skipped / 2026-11-12T17:25:02Z
- Parts-Plan-Write / skipped / 2026-11-12T17:25:03Z
