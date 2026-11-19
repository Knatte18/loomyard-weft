loom halted blocked at Finalize

```
the task worktree has uncommitted changes before the merge-in: internal/hardenshed/.lock; commit or remove them on the task branch, then resume
```

producer: Finalize
state: blocked
anomaly: escalation-to-human
history_entries: 29
trace_file: /home/knatte/Code/loomyard-LYMHUB/hardener-campaign/.lym/logs/trace-20261119T174114Z-3118160fff8e99b5-2063193.log

history:
- Preflight / done / 2026-11-19T13:29:06Z
- Motif / skipped / 2026-11-19T13:29:13Z
- Drift-Baseline / skipped / 2026-11-19T13:29:21Z
- Parts-Write / skipped / 2026-11-19T13:29:31Z
- Parts-Plan-Write / skipped / 2026-11-19T13:29:36Z
- Plan-Write / done / 2026-11-19T13:57:50Z
- Plan-Bouncer / stuck / 2026-11-19T13:58:14Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-19T14:10:29Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-19T14:18:10Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-19T14:18:44Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-19T14:21:05Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-19T14:23:46Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-19T14:24:18Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-19T14:26:24Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-19T14:26:44Z / routed to Plan-Bouncer
- Plan-Bouncer / done / 2026-11-19T14:27:10Z
- Drift-Await / skipped / 2026-11-19T14:27:17Z
- Drift-Expect / skipped / 2026-11-19T14:27:22Z
- Webster / done / 2026-11-19T16:52:19Z
- Parent-Align / done / 2026-11-19T17:02:23Z
- Drift-Verify / skipped / 2026-11-19T17:02:34Z
- Code-Bouncer / stuck / 2026-11-19T17:03:10Z / routed to Code-Burler-Review
- Code-Burler-Review / stuck / 2026-11-19T17:10:46Z / routed to Code-Burler-Fix
- Code-Burler-Fix / stuck / 2026-11-19T17:39:24Z / routed to Code-Bouncer
- Code-Bouncer / done / 2026-11-19T17:40:00Z
- Drift-Recheck / skipped / 2026-11-19T17:40:09Z
- Describe / done / 2026-11-19T17:40:53Z
- PR-Review / done / 2026-11-19T17:41:00Z
- Finalize / stuck / 2026-11-19T17:41:18Z
