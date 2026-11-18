loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1 attempt 1: review: burler: round reached done but its review file is invalid: burler: review file frontmatter is not valid YAML: yaml: line 23: mapping values are not allowed in this context
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/import-loses-issue-evidence/.lym/logs/trace-20261118T190810Z-21b0c0d4e1e17fb1-977009.log

history:
- Preflight / done / 2026-11-18T18:47:57Z
- Record-Write / done / 2026-11-18T18:48:12Z
- Drift-Baseline / skipped / 2026-11-18T18:48:17Z
- Drift-Await / skipped / 2026-11-18T18:48:22Z
- Drift-Expect / skipped / 2026-11-18T18:48:27Z
- Darn-Write / done / 2026-11-18T19:07:04Z
- Drift-Verify / skipped / 2026-11-18T19:07:18Z
- Code-Bouncer / stuck / 2026-11-18T19:08:04Z
