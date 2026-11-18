loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1 attempt 1: review: burler: round reached done but its review file is invalid: burler: review file frontmatter is not valid YAML: yaml: line 8: mapping values are not allowed in this context
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/board-view-not-refreshing/.lym/logs/trace-20261118T185423Z-d7f7356879ac469c-4040979.log

history:
- Preflight / done / 2026-11-18T18:45:31Z
- Record-Write / done / 2026-11-18T18:45:36Z
- Drift-Baseline / skipped / 2026-11-18T18:45:41Z
- Drift-Await / skipped / 2026-11-18T18:45:46Z
- Drift-Expect / skipped / 2026-11-18T18:45:51Z
- Darn-Write / done / 2026-11-18T18:53:40Z
- Drift-Verify / skipped / 2026-11-18T18:53:51Z
- Code-Bouncer / stuck / 2026-11-18T18:54:18Z
