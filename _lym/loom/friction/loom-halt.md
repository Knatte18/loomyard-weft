loom halted blocked at Finalize

```
landing stop: the fixer session of Finalize pushed origin/pr-modes-no-agent at b6eb9ac69a8d5e12c863dd0a5a27587e1ff0446a; session log: /home/knatte/Code/loomyard-LYMHUB/pr-modes-no-agent/.lym/shuttle/55336935c2b090f90010928604d00408; way forward: check the pushed commits are the task's, then `lym loom resume`
```

producer: Finalize
state: blocked
anomaly: escalation-to-human
history_entries: 18
trace_file: /home/knatte/Code/loomyard-LYMHUB/pr-modes-no-agent/.lym/logs/trace-20261126T212926Z-bb655e9b08ce7f19-2965360.log

history:
- Preflight / done / 2026-11-26T18:51:10Z
- Record-Write / done / 2026-11-26T18:51:10Z
- Drift-Baseline / skipped / 2026-11-26T18:51:11Z
- Drift-Await / skipped / 2026-11-26T18:51:11Z
- Drift-Expect / skipped / 2026-11-26T18:51:11Z
- Darn-Write / done / 2026-11-26T19:58:41Z
- Drift-Verify / skipped / 2026-11-26T19:58:45Z
- Code-Bouncer / stuck / 2026-11-26T19:59:57Z / routed to Code-Burler-Review
- Code-Burler-Review / stuck / 2026-11-26T20:10:52Z / routed to Code-Burler-Fix
- Code-Burler-Fix / stuck / 2026-11-26T21:05:06Z / routed to Code-Bouncer
- Code-Bouncer / stuck / 2026-11-26T21:05:59Z / routed to Code-Burler-Review
- Code-Burler-Review / stuck / 2026-11-26T21:11:41Z / routed to Code-Burler-Fix
- Code-Burler-Fix / stuck / 2026-11-26T21:27:40Z / routed to Code-Bouncer
- Code-Bouncer / done / 2026-11-26T21:28:24Z
- Drift-Recheck / skipped / 2026-11-26T21:28:28Z
- Describe / done / 2026-11-26T21:29:21Z
- PR-Review / done / 2026-11-26T21:29:26Z
- Finalize / stuck / 2026-11-27T06:33:53Z
