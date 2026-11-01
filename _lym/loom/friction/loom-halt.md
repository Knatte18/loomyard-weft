loom halted blocked at Finalize

```
the task worktree has uncommitted changes before the merge-in: 200k, 250k; commit or remove them on the task branch, then resume
```

producer: Finalize
state: blocked
anomaly: escalation-to-human
history_entries: 10
trace_file: /home/knatte/Code/loomyard-LYMHUB/batch-budget-300k-trial/.lym/logs/trace-20261101T114004Z-5bfe28a07d5833d9-930346.log

history:
- Preflight / done / 2026-11-01T11:35:46Z
- Record-Write / done / 2026-11-01T11:35:46Z
- Drift-Baseline / skipped / 2026-11-01T11:35:46Z
- Drift-Await / skipped / 2026-11-01T11:35:47Z
- Darn-Write / done / 2026-11-01T11:40:02Z
- Drift-Verify / skipped / 2026-11-01T11:40:03Z
- Code-Bouncer / done / 2026-11-01T11:40:04Z
- Describe / skipped / 2026-11-01T11:40:04Z
- PR-Review / done / 2026-11-01T11:40:04Z
- Finalize / stuck / 2026-11-01T11:40:07Z
