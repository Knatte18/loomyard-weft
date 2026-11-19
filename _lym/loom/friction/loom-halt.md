loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1 attempt 1: review: burler: round reached done but its review file is invalid: burler: review file frontmatter is not valid YAML: yaml: line 22: mapping values are not allowed in this context
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/creel-slack-post-silent-when-off/.lym/logs/trace-20261119T095419Z-390768d9d492cd2e-1887766.log

history:
- Preflight / done / 2026-11-19T09:28:39Z
- Record-Write / done / 2026-11-19T09:28:45Z
- Drift-Baseline / skipped / 2026-11-19T09:28:54Z
- Drift-Await / skipped / 2026-11-19T09:29:02Z
- Drift-Expect / skipped / 2026-11-19T09:29:08Z
- Darn-Write / done / 2026-11-19T09:53:23Z
- Drift-Verify / skipped / 2026-11-19T09:53:35Z
- Code-Bouncer / stuck / 2026-11-19T09:54:10Z
