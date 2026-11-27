loom halted failed at Finalize

```
landingshed: Finalize: read GitHub's rules for the parent branch: pullrequest: read the rules active on branch main: Get "https://api.github.com/repos/Knatte18/loomyard/rules/branches/main?per_page=100": dial tcp: lookup api.github.com on 127.0.0.53:53: server misbehaving
```

producer: Finalize
state: failed
anomaly: producer-hard-failure
history_entries: 14
trace_file: /home/knatte/Code/loomyard-LYMHUB/reach-misses-tier1-calls/.lym/logs/trace-20261126T214016Z-94c34557f932d8f7-3655111.log

history:
- Preflight / done / 2026-11-26T20:12:29Z
- Record-Write / done / 2026-11-26T20:12:30Z
- Drift-Baseline / skipped / 2026-11-26T20:12:30Z
- Drift-Await / skipped / 2026-11-26T20:12:30Z
- Drift-Expect / skipped / 2026-11-26T20:12:31Z
- Darn-Write / done / 2026-11-26T21:14:02Z
- Drift-Verify / skipped / 2026-11-26T21:14:04Z
- Code-Bouncer / stuck / 2026-11-26T21:14:41Z / routed to Code-Burler-Review
- Code-Burler-Review / stuck / 2026-11-26T21:21:02Z / routed to Code-Burler-Fix
- Code-Burler-Fix / stuck / 2026-11-26T21:38:15Z / routed to Code-Bouncer
- Code-Bouncer / done / 2026-11-26T21:39:04Z
- Drift-Recheck / skipped / 2026-11-26T21:39:18Z
- Describe / done / 2026-11-26T21:40:13Z
- PR-Review / done / 2026-11-26T21:40:15Z
