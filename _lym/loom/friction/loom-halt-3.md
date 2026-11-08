loom halted failed at Darn-Write

```
loop-exited: loop f0ba24134134d160 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/shed/slack-presence-auto/steps/loop.log; step Darn-Write was in flight under trace d11bda90ba9b54b4; loop exit: not recorded
status error: shedverbs: step Darn-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step slack-presence-auto --until-stop
```

producer: Darn-Write
state: failed
anomaly: interrupted-stop
history_entries: 7
trace_file: /home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/logs/trace-20261108T143801Z-d11bda90ba9b54b4-14690.log

history:
- Preflight / done / 2026-11-08T13:07:09Z
- Record-Write / done / 2026-11-08T13:07:11Z
- Drift-Baseline / skipped / 2026-11-08T13:07:14Z
- Drift-Await / skipped / 2026-11-08T13:07:18Z
- Drift-Expect / skipped / 2026-11-08T13:07:19Z
- Darn-Write / stuck / 2026-11-08T14:37:19Z
- Darn-Write / goto / 2026-11-08T14:38:01Z
