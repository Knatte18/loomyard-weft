loom halted blocked at Darn-Write

```
Darn-Write (seats): the chair's session timed out and was stopped; elapsed 3h0m2s, limit 3h0m0s, last session state busy (cause background); way forward: run lym loom resume, which starts the row over in a fresh session
```

producer: Darn-Write
state: blocked
anomaly: escalation-to-human
history_entries: 6
trace_file: /home/knatte/Code/loomyard-LYMHUB/weaver-bench-rerun/.lym/logs/trace-20261128T134948Z-d45d17b2c502df88-2498958.log

history:
- Preflight / done / 2026-11-28T13:49:46Z
- Record-Write / done / 2026-11-28T13:49:48Z
- Drift-Baseline / skipped / 2026-11-28T13:49:49Z
- Drift-Await / skipped / 2026-11-28T13:49:49Z
- Drift-Expect / skipped / 2026-11-28T13:49:49Z
- Darn-Write / stuck / 2026-11-28T16:49:53Z
