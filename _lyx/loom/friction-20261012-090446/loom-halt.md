loom halted blocked at Publish

```
verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/test-tmp-cleanup/.lyx/verify/verify-4.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 23
trace_file: /home/knatte/Code/loomyard-LYXHUB/test-tmp-cleanup/.lyx/logs/trace-20261012T084430Z-c3c970b1612422b8-1315230.log

history:
- Preflight / done / 2026-10-12T07:00:16Z
- Loom-Preflight / done / 2026-10-12T07:00:19Z
- Discussion-Write / done / 2026-10-12T07:10:07Z
- Discussion-Bouncer / stuck / 2026-10-12T07:10:47Z
- Discussion-Burler-Review / stuck / 2026-10-12T07:17:14Z
- Discussion-Burler-Fix / stuck / 2026-10-12T07:19:56Z
- Discussion-Bouncer / stuck / 2026-10-12T07:20:34Z
- Discussion-Burler-Review / stuck / 2026-10-12T07:25:10Z
- Discussion-Burler-Fix / stuck / 2026-10-12T07:26:26Z
- Discussion-Bouncer / done / 2026-10-12T07:27:15Z
- Plan-Write / done / 2026-10-12T07:52:47Z
- Plan-Bouncer / stuck / 2026-10-12T07:53:13Z
- Plan-Burler-Review / stuck / 2026-10-12T08:00:44Z
- Plan-Burler-Fix / stuck / 2026-10-12T08:02:45Z
- Plan-Bouncer / done / 2026-10-12T08:03:23Z
- Batchifier / done / 2026-10-12T08:03:34Z
- Webster / done / 2026-10-12T08:36:03Z
- Webster-Bouncer / stuck / 2026-10-12T08:36:36Z
- Webster-Burler-Review / stuck / 2026-10-12T08:39:18Z
- Webster-Burler-Fix / stuck / 2026-10-12T08:43:09Z
- Webster-Bouncer / done / 2026-10-12T08:43:37Z
- Describe / done / 2026-10-12T08:44:18Z
- Publish / stuck / 2026-10-12T08:46:42Z
