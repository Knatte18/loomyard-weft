loom halted blocked at Code-Bouncer

```
bounce budget exhausted for Code-Bouncer; way forward: run "lym loom resume" in the task worktree: the resume grants the row one more budget; a goto keeps the spent budget and grants none
```

producer: Code-Bouncer
state: blocked
anomaly: bounce-budget-exhausted
history_entries: 11
trace_file: /home/knatte/Code/loomyard-LYMHUB/pr-findings-open-fabric-nil/.lym/logs/trace-20261123T091309Z-627eb647532e5596-3167728.log

history:
- Preflight / done / 2026-11-23T08:22:07Z
- Record-Write / done / 2026-11-23T08:22:49Z
- Drift-Baseline / skipped / 2026-11-23T08:23:22Z
- Drift-Await / skipped / 2026-11-23T08:23:54Z
- Drift-Expect / skipped / 2026-11-23T08:24:23Z
- Darn-Write / done / 2026-11-23T08:38:29Z
- Drift-Verify / skipped / 2026-11-23T08:38:44Z
- Code-Bouncer / stuck / 2026-11-23T08:47:25Z / routed to Code-Burler-Review
- Code-Burler-Review / stuck / 2026-11-23T09:09:40Z / routed to Code-Burler-Fix
- Code-Burler-Fix / stuck / 2026-11-23T09:12:49Z / routed to Code-Bouncer
- Code-Bouncer / stuck / 2026-11-23T09:21:25Z
