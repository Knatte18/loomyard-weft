loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/viewer-off-shell/.lyx/verify/verify-5.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 31
trace_file: /home/knatte/Code/loomyard-LYXHUB/viewer-off-shell/.lyx/logs/trace-20261015T124434Z-f9f903d495b43124-2704961.log

history:
- Preflight / done / 2026-10-15T11:28:15Z
- Loom-Preflight / done / 2026-10-15T11:28:22Z
- Discussion-Write / done / 2026-10-15T11:33:47Z
- Discussion-Bouncer / stuck / 2026-10-15T11:34:21Z
- Discussion-Burler-Review / stuck / 2026-10-15T11:37:30Z
- Discussion-Burler-Fix / stuck / 2026-10-15T11:39:04Z
- Discussion-Bouncer / stuck / 2026-10-15T11:39:44Z
- Discussion-Burler-Review / stuck / 2026-10-15T11:42:14Z
- Discussion-Burler-Fix / stuck / 2026-10-15T11:42:55Z
- Discussion-Bouncer / done / 2026-10-15T11:43:23Z
- Plan-Write / done / 2026-10-15T11:59:13Z
- Plan-Bouncer / stuck / 2026-10-15T11:59:44Z
- Plan-Burler-Review / stuck / 2026-10-15T12:09:07Z
- Plan-Burler-Fix / stuck / 2026-10-15T12:10:32Z
- Plan-Bouncer / stuck / 2026-10-15T12:11:03Z
- Plan-Burler-Review / stuck / 2026-10-15T12:16:19Z
- Plan-Burler-Fix / stuck / 2026-10-15T12:18:04Z
- Plan-Bouncer / done / 2026-10-15T12:19:24Z
- Batchifier / done / 2026-10-15T12:19:30Z
- Webster / done / 2026-10-15T12:37:18Z
- Webster-Bouncer / stuck / 2026-10-15T12:37:51Z
- Webster-Burler-Review / stuck / 2026-10-15T12:39:43Z
- Webster-Burler-Fix / stuck / 2026-10-15T12:40:06Z
- Webster-Bouncer / done / 2026-10-15T12:40:36Z
- Describe / done / 2026-10-15T12:41:14Z
- Merge-In / done / 2026-10-15T12:41:48Z
- Merge-Bouncer / stuck / 2026-10-15T12:42:15Z
- Merge-Burler-Review / stuck / 2026-10-15T12:43:33Z
- Merge-Burler-Fix / stuck / 2026-10-15T12:43:53Z
- Merge-Bouncer / done / 2026-10-15T12:44:23Z
- Publish / stuck / 2026-10-15T12:46:01Z
