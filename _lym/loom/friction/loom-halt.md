loom halted blocked at Webster

```
run-exit audit found 1 correctness finding(s): 1) parent-write: Master wrote "/home/knatte/Code/loomyard-LYMHUB/kde-window-placement-build/_lym/loom/friction/webster-master.md" — Master may write only its two contract files (outcome.yaml and summary.md) and its friction note; any other write means Master implemented a batch itself or hand-wrote a batch report (/home/knatte/Code/loomyard-LYMHUB/kde-window-placement-build/_lym/loom/friction/webster-master.md: changed since Master spawned); way forward: 1) restore /home/knatte/Code/loomyard-LYMHUB/kde-window-placement-build/_lym/loom/friction/webster-master.md to records-side commit 6bbba30721e6f1773da5a26e11b11a86e9fa5878 (rm a path that commit does not hold); 2) lym webster accept-audit; 3) re-step the Webster row
```

producer: Webster
state: blocked
anomaly: escalation-to-human
history_entries: 19
trace_file: /home/knatte/Code/loomyard-LYMHUB/kde-window-placement-build/.lym/logs/trace-20261127T155245Z-a98b46f77fbb1d30-962194.log

history:
- Preflight / done / 2026-11-27T14:35:18Z
- Motif / skipped / 2026-11-27T14:35:19Z
- Drift-Baseline / skipped / 2026-11-27T14:35:19Z
- Parts-Write / skipped / 2026-11-27T14:35:19Z
- Parts-Plan-Write / skipped / 2026-11-27T14:35:19Z
- Plan-Write / done / 2026-11-27T14:58:34Z
- Plan-Bouncer / stuck / 2026-11-27T14:59:17Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-27T15:11:51Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-27T15:24:10Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-27T15:24:55Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-27T15:31:37Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-27T15:44:50Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-27T15:45:25Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-27T15:52:07Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-27T15:52:19Z / routed to Plan-Bouncer
- Plan-Bouncer / done / 2026-11-27T15:52:45Z
- Drift-Await / skipped / 2026-11-27T15:52:46Z
- Drift-Expect / skipped / 2026-11-27T15:52:46Z
- Webster / stuck / 2026-11-28T13:31:55Z
