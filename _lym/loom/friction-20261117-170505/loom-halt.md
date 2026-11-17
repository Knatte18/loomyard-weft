loom halted failed at Code-Burler-Fix

```
shedadapters: Code-Burler-Fix (burler-fix): round 1 attempt 1: fix: burler: twill fixer fork "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYMHUB-leftover-pairs-teardown/f64804ff-1459-4477-ac4f-6ffbbae2d279/subagents/agent-a654e39025dc97419.jsonl" attempted 5 write/edit tool call(s) — a fork must never mutate a file; way forward: inspect the target's working-tree changes, revert what the fork wrote, and resume the run
```

producer: Code-Burler-Fix
state: failed
anomaly: producer-hard-failure
history_entries: 9
trace_file: /home/knatte/Code/loomyard-LYMHUB/leftover-pairs-teardown/.lym/logs/trace-20261117T162737Z-f007c94463868f39-2847149.log

history:
- Preflight / done / 2026-11-17T15:52:38Z
- Record-Write / done / 2026-11-17T15:52:43Z
- Drift-Baseline / skipped / 2026-11-17T15:52:45Z
- Drift-Await / skipped / 2026-11-17T15:52:52Z
- Drift-Expect / skipped / 2026-11-17T15:52:59Z
- Darn-Write / done / 2026-11-17T16:20:37Z
- Drift-Verify / skipped / 2026-11-17T16:20:40Z
- Code-Bouncer / stuck / 2026-11-17T16:20:58Z
- Code-Burler-Review / stuck / 2026-11-17T16:27:33Z
