loom halted blocked at Darn-Write

```
Darn-Write (seats): the chair's session timed out; elapsed 3h0m1s, limit 3h0m0s, last session state busy (cause turn); way forward: run lym loom resume, which re-attaches the session while it is live and otherwise starts the row over
```

producer: Darn-Write
state: blocked
anomaly: escalation-to-human
history_entries: 6
trace_file: /home/knatte/Code/loomyard-LYMHUB/weaver-bench/.lym/logs/trace-20261127T115007Z-5b941df07b151b75-2325816.log

history:
- Preflight / done / 2026-11-27T11:50:06Z
- Record-Write / done / 2026-11-27T11:50:07Z
- Drift-Baseline / skipped / 2026-11-27T11:50:08Z
- Drift-Await / skipped / 2026-11-27T11:50:08Z
- Drift-Expect / skipped / 2026-11-27T11:50:08Z
- Darn-Write / stuck / 2026-11-27T14:50:10Z
