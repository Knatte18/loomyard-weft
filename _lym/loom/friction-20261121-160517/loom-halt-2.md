loom halted blocked at Webster

```
Batch 01-gitrepo-refspec-upstream: recover-batch refused with batch_failed: create-not-done/1-gitrepo-refspec-upstream[blocking]: Create target "plan:internal/gitrepo#remoteTrackingRef" still does not resolve; the fork reverted the refspec-following reads under the superseded no-merge-config decision, which the parent's later Option C decision (#2210) reverses.
```

producer: Webster
state: blocked
anomaly: escalation-to-human
history_entries: 16
trace_file: /home/knatte/Code/loomyard-LYMHUB/single-repo-namespace-layout/.lym/logs/trace-20261121T141049Z-8a209421582530b7-844191.log

history:
- Preflight / done / 2026-11-19T19:34:51Z
- Motif / skipped / 2026-11-19T19:35:03Z
- Drift-Baseline / skipped / 2026-11-19T19:35:10Z
- Parts-Write / skipped / 2026-11-19T19:35:22Z
- Parts-Plan-Write / skipped / 2026-11-19T19:35:34Z
- Plan-Write / done / 2026-11-19T20:02:17Z
- Plan-Bouncer / stuck / 2026-11-21T13:28:09Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-21T13:43:21Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-21T13:57:09Z / routed to Plan-Bouncer
- Plan-Bouncer / stuck / 2026-11-21T13:57:48Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-21T14:05:49Z / routed to Plan-Burler-Fix
- Plan-Burler-Fix / stuck / 2026-11-21T14:10:09Z / routed to Plan-Bouncer
- Plan-Bouncer / done / 2026-11-21T14:10:42Z
- Drift-Await / skipped / 2026-11-21T14:10:49Z
- Drift-Expect / skipped / 2026-11-21T14:10:54Z
- Webster / stuck / 2026-11-21T14:32:53Z
