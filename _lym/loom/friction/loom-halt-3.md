loom halted blocked at Webster

```
webster: correctness findings from an earlier run exit are pending: 1 correctness finding(s): 1) parent-write: Master wrote "/home/knatte/Code/loomyard-LYMHUB/config-keys-nested/_lym/loom/friction/webster-master.md" — Master may write only its two contract files (outcome.yaml and summary.md); any other write means Master implemented a batch itself or hand-wrote a batch report (/home/knatte/Code/loomyard-LYMHUB/config-keys-nested/_lym/loom/friction/webster-master.md: cannot be checked: outside the task worktree); way forward: 1) lym webster reset --to start; 2) re-step the Webster row
```

producer: Webster
state: blocked
anomaly: escalation-to-human
history_entries: 20
trace_file: /home/knatte/Code/loomyard-LYMHUB/config-keys-nested/.lym/logs/trace-20261126T131056Z-d48e83d96747e72d-3020738.log

history:
- Preflight / done / 2026-11-26T08:58:06Z
- Motif / skipped / 2026-11-26T08:58:06Z
- Drift-Baseline / skipped / 2026-11-26T08:58:06Z
- Parts-Write / skipped / 2026-11-26T08:58:06Z
- Parts-Plan-Write / skipped / 2026-11-26T08:58:06Z
- Plan-Write / done / 2026-11-26T09:19:04Z
- Plan-Bouncer / stuck / 2026-11-26T09:20:15Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-26T09:29:17Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-26T09:37:03Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-26T09:37:45Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-26T09:49:15Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-26T09:54:21Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-26T09:54:56Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-26T10:03:29Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-26T10:03:42Z / routed to Plan-Bouncer
- Plan-Bouncer / done / 2026-11-26T10:04:15Z
- Drift-Await / skipped / 2026-11-26T10:04:16Z
- Drift-Expect / skipped / 2026-11-26T10:04:16Z
- Webster / stuck / 2026-11-26T13:02:22Z
- Webster / stuck / 2026-11-26T13:02:26Z
