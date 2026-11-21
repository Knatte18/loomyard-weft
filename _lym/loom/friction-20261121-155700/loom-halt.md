loom halted blocked at Darn-Write

```
gate did not pass after 4 attempts; findings: /home/knatte/Code/loomyard-LYMHUB/red-first-overlay-non-go-files/.lym/shed/red-first-overlay-non-go-files/gate-findings/Darn-Write.md; way forward: run "lym loom resume" in the task worktree, which re-spawns Darn-Write with a fresh verify budget
```

producer: Darn-Write
state: blocked
anomaly: escalation-to-human
history_entries: 6
trace_file: /home/knatte/Code/loomyard-LYMHUB/red-first-overlay-non-go-files/.lym/logs/trace-20261121T151329Z-4963b7185fe26770-3190074.log

history:
- Preflight / done / 2026-11-21T15:13:16Z
- Record-Write / done / 2026-11-21T15:13:23Z
- Drift-Baseline / skipped / 2026-11-21T15:13:29Z
- Drift-Await / skipped / 2026-11-21T15:13:34Z
- Drift-Expect / skipped / 2026-11-21T15:13:40Z
- Darn-Write / stuck / 2026-11-21T15:32:47Z
