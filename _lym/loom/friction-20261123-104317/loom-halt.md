loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1 attempt 1b: review: stencilstore: stencil "burler-assist-consolidation" was written by lym build 8cf7f477563f044e178a070cfd7dd27cb9f03c78, which is newer than the running build 52d132a2c8e4e3b68e3e797e30d3e9ceb200ec50, and this process may not render it; way forward: restart the run's driver on the current lym binary, and update that binary first if it is older than the board
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/godocreflow-worsens-findings/.lym/logs/trace-20261123T090022Z-9248a55650778889-2307740.log

history:
- Preflight / done / 2026-11-23T08:25:29Z
- Record-Write / done / 2026-11-23T08:25:45Z
- Drift-Baseline / skipped / 2026-11-23T08:26:07Z
- Drift-Await / skipped / 2026-11-23T08:26:23Z
- Drift-Expect / skipped / 2026-11-23T08:26:30Z
- Darn-Write / done / 2026-11-23T08:51:02Z
- Drift-Verify / skipped / 2026-11-23T08:51:13Z
- Code-Bouncer / stuck / 2026-11-23T08:59:59Z / routed to Code-Burler-Review
