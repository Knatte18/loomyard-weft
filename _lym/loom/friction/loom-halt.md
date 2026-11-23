loom halted blocked at Code-Bouncer

```
bounce budget exhausted for Code-Bouncer; way forward: run "lym loom resume" in the task worktree: the resume grants the row one more budget; a goto keeps the spent budget and grants none
```

producer: Code-Bouncer
state: blocked
anomaly: bounce-budget-exhausted
history_entries: 11
trace_file: /home/knatte/Code/loomyard-LYMHUB/lint-comments-reads-head/.lym/logs/trace-20261123T111134Z-058d789805f0f91c-2555274.log

history:
- Preflight / done / 2026-11-23T10:44:45Z
- Record-Write / done / 2026-11-23T10:45:17Z
- Drift-Baseline / skipped / 2026-11-23T10:45:29Z
- Drift-Await / skipped / 2026-11-23T10:45:40Z
- Drift-Expect / skipped / 2026-11-23T10:46:17Z
- Darn-Write / done / 2026-11-23T10:58:12Z
- Drift-Verify / skipped / 2026-11-23T10:58:41Z
- Code-Bouncer / stuck / 2026-11-23T10:59:51Z / routed to Code-Burler-Review
- Code-Burler-Review / stuck / 2026-11-23T11:02:57Z / routed to Code-Burler-Fix
- Code-Burler-Fix / stuck / 2026-11-23T11:11:03Z / routed to Code-Bouncer
- Code-Bouncer / stuck / 2026-11-23T11:11:34Z
