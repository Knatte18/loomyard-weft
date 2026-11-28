loom halted blocked at Finalize

```
landing stop: the remote ref origin/gatecli-dotnet-redfirst-time moved during the fixer session of Finalize to 9663d3073f3871d95b752bf994a232e691862213, which holds commits the task head 8169ba2b6d932c03be922f42986da6419d95521e the session started from lacks; session: claude session 48927808-ca1f-49c4-b067-366f4f2b2afa; way forward: check the commits 8169ba2b6d932c03be922f42986da6419d95521e..9663d3073f3871d95b752bf994a232e691862213 are the task's, then `lym loom resume`
```

producer: Finalize
state: blocked
anomaly: escalation-to-human
history_entries: 15
trace_file: /home/knatte/Code/loomyard-LYMHUB/gatecli-dotnet-redfirst-time/.lym/logs/trace-20261128T104307Z-7cbfcfd30afe0ceb-2254283.log

history:
- Preflight / done / 2026-11-28T10:11:45Z
- Record-Write / done / 2026-11-28T10:11:47Z
- Drift-Baseline / skipped / 2026-11-28T10:11:48Z
- Drift-Await / skipped / 2026-11-28T10:11:48Z
- Drift-Expect / skipped / 2026-11-28T10:11:48Z
- Darn-Write / done / 2026-11-28T10:27:23Z
- Drift-Verify / skipped / 2026-11-28T10:27:24Z
- Code-Bouncer / stuck / 2026-11-28T10:28:00Z / routed to Code-Burler-Review
- Code-Burler-Review / stuck / 2026-11-28T10:31:38Z / routed to Code-Burler-Fix
- Code-Burler-Fix / stuck / 2026-11-28T10:40:39Z / routed to Code-Bouncer
- Code-Bouncer / done / 2026-11-28T10:41:25Z
- Drift-Recheck / skipped / 2026-11-28T10:41:29Z
- Describe / done / 2026-11-28T10:43:03Z
- PR-Review / done / 2026-11-28T10:43:06Z
- Finalize / stuck / 2026-11-28T11:31:37Z
