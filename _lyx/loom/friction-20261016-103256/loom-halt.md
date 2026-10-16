loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/reed-add-parentless-placement/.lyx/verify/verify-5.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 31
trace_file: /home/knatte/Code/loomyard-LYXHUB/reed-add-parentless-placement/.lyx/logs/trace-20261016T100611Z-f8390f89d953fa25-3466204.log

history:
- Preflight / done / 2026-10-16T07:54:28Z
- Loom-Preflight / done / 2026-10-16T07:54:40Z
- Discussion-Write / done / 2026-10-16T08:03:51Z
- Discussion-Bouncer / stuck / 2026-10-16T08:04:19Z
- Discussion-Burler-Review / stuck / 2026-10-16T08:08:47Z
- Discussion-Burler-Fix / stuck / 2026-10-16T08:10:04Z
- Discussion-Bouncer / stuck / 2026-10-16T08:10:43Z
- Discussion-Burler-Review / stuck / 2026-10-16T08:13:33Z
- Discussion-Burler-Fix / stuck / 2026-10-16T08:14:30Z
- Discussion-Bouncer / done / 2026-10-16T08:15:17Z
- Plan-Write / done / 2026-10-16T08:43:18Z
- Plan-Bouncer / stuck / 2026-10-16T08:43:50Z
- Plan-Burler-Review / stuck / 2026-10-16T08:52:09Z
- Plan-Burler-Fix / stuck / 2026-10-16T08:53:31Z
- Plan-Bouncer / stuck / 2026-10-16T08:54:09Z
- Plan-Burler-Review / stuck / 2026-10-16T09:00:45Z
- Plan-Burler-Fix / stuck / 2026-10-16T09:02:04Z
- Plan-Bouncer / done / 2026-10-16T09:02:40Z
- Batchifier / done / 2026-10-16T09:02:52Z
- Webster / done / 2026-10-16T09:57:20Z
- Webster-Bouncer / stuck / 2026-10-16T09:57:53Z
- Webster-Burler-Review / stuck / 2026-10-16T09:59:08Z
- Webster-Burler-Fix / stuck / 2026-10-16T09:59:33Z
- Webster-Bouncer / done / 2026-10-16T10:00:11Z
- Describe / done / 2026-10-16T10:00:51Z
- Merge-In / done / 2026-10-16T10:02:39Z
- Merge-Bouncer / stuck / 2026-10-16T10:03:18Z
- Merge-Burler-Review / stuck / 2026-10-16T10:04:22Z
- Merge-Burler-Fix / stuck / 2026-10-16T10:05:32Z
- Merge-Bouncer / done / 2026-10-16T10:06:00Z
- Publish / stuck / 2026-10-16T10:07:28Z
