loom halted blocked at Darn-Write

```
gate did not pass after 0 attempts; findings: /home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/shed/slack-presence-auto/gate-findings/Darn-Write.md; way forward: run "lym loom resume" in the task worktree, which re-spawns Darn-Write with a fresh verify budget
```

producer: Darn-Write
state: blocked
anomaly: escalation-to-human
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/logs/trace-20261108T144749Z-2d9cd3e98d81afc9-6244.log

history:
- Preflight / done / 2026-11-08T13:07:09Z
- Record-Write / done / 2026-11-08T13:07:11Z
- Drift-Baseline / skipped / 2026-11-08T13:07:14Z
- Drift-Await / skipped / 2026-11-08T13:07:18Z
- Drift-Expect / skipped / 2026-11-08T13:07:19Z
- Darn-Write / stuck / 2026-11-08T14:37:19Z
- Darn-Write / goto / 2026-11-08T14:38:01Z
- Darn-Write / stuck / 2026-11-08T15:04:45Z
