loom halted failed at Plan-Burler-Fix

```
shedadapters: Plan-Burler-Fix (burler-fix): round 2 attempt 2: fix: burler: twill fixer fork "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYMHUB-fabric-core-foundation/fecd010a-97a1-415c-8d4a-716a0e7703e8/subagents/agent-abc5528dbce58b86c.jsonl" attempted 4 nested spawn call(s) — forks cannot nest and must never spawn a fork or subagent, even when the attempt was denied; no target change was found: no fork wrote a file or ran a state-changing git command outside a repository it created; way forward: resume the run
```

producer: Plan-Burler-Fix
state: failed
anomaly: producer-hard-failure
history_entries: 11
trace_file: /home/knatte/Code/loomyard-LYMHUB/fabric-core-foundation/.lym/logs/trace-20261127T074702Z-03a814563ec5ecdb-2432246.log

history:
- Preflight / done / 2026-11-27T06:27:37Z
- Motif / skipped / 2026-11-27T06:27:40Z
- Drift-Baseline / skipped / 2026-11-27T06:27:40Z
- Parts-Write / skipped / 2026-11-27T06:27:40Z
- Parts-Plan-Write / skipped / 2026-11-27T06:27:41Z
- Plan-Write / done / 2026-11-27T07:05:05Z
- Plan-Bouncer / stuck / 2026-11-27T07:06:04Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-27T07:22:48Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-27T07:36:17Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-27T07:36:55Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-27T07:47:02Z / routed to Plan-Burler-Fix
