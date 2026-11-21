loom halted blocked at Darn-Write

```
gate did not pass after 3 attempts; findings: /home/knatte/Code/loomyard-LYMHUB/gate-dotnet-fixes/.lym/shed/gate-dotnet-fixes/gate-findings/Darn-Write.md; way forward: run "lym loom resume" in the task worktree, which re-spawns Darn-Write with a fresh verify budget
```

producer: Darn-Write
state: blocked
anomaly: escalation-to-human
history_entries: 6
trace_file: /home/knatte/Code/loomyard-LYMHUB/gate-dotnet-fixes/.lym/logs/trace-20261119T194526Z-98cc2e1d29d8000b-474981.log

history:
- Preflight / done / 2026-11-19T19:45:06Z
- Record-Write / done / 2026-11-19T19:45:17Z
- Drift-Baseline / skipped / 2026-11-19T19:45:27Z
- Drift-Await / skipped / 2026-11-19T19:45:44Z
- Drift-Expect / skipped / 2026-11-19T19:45:52Z
- Darn-Write / stuck / 2026-11-21T13:37:22Z
