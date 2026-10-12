loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/merge-review/.lyx/verify/verify-4.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 30
trace_file: /home/knatte/Code/loomyard-LYXHUB/merge-review/.lyx/logs/trace-20261012T093930Z-1f387318933e2e77-2903934.log

history:
- Preflight / done / 2026-10-12T07:00:10Z
- Loom-Preflight / done / 2026-10-12T07:00:16Z
- Discussion-Write / done / 2026-10-12T07:11:22Z
- Discussion-Bouncer / stuck / 2026-10-12T07:11:56Z
- Discussion-Burler-Review / stuck / 2026-10-12T07:17:33Z
- Discussion-Burler-Fix / stuck / 2026-10-12T07:19:07Z
- Discussion-Bouncer / stuck / 2026-10-12T07:19:48Z
- Discussion-Burler-Review / stuck / 2026-10-12T07:24:16Z
- Discussion-Burler-Fix / stuck / 2026-10-12T07:25:34Z
- Discussion-Bouncer / awaiting / 2026-10-12T07:26:15Z
- Discussion-Bouncer / done / 2026-10-12T07:26:59Z
- Plan-Write / done / 2026-10-12T07:55:09Z
- Plan-Bouncer / stuck / 2026-10-12T07:55:55Z
- Plan-Burler-Review / stuck / 2026-10-12T08:03:53Z
- Plan-Burler-Fix / stuck / 2026-10-12T08:07:00Z
- Plan-Bouncer / stuck / 2026-10-12T08:07:37Z
- Plan-Burler-Review / stuck / 2026-10-12T08:11:28Z
- Plan-Burler-Fix / stuck / 2026-10-12T08:13:09Z
- Plan-Bouncer / stuck / 2026-10-12T08:13:40Z
- Plan-Burler-Review / stuck / 2026-10-12T08:16:16Z
- Plan-Burler-Fix / stuck / 2026-10-12T08:17:39Z
- Plan-Bouncer / done / 2026-10-12T08:18:21Z
- Batchifier / done / 2026-10-12T08:18:26Z
- Webster / done / 2026-10-12T09:33:10Z
- Webster-Bouncer / stuck / 2026-10-12T09:33:41Z
- Webster-Burler-Review / stuck / 2026-10-12T09:36:06Z
- Webster-Burler-Fix / stuck / 2026-10-12T09:38:13Z
- Webster-Bouncer / done / 2026-10-12T09:38:37Z
- Describe / done / 2026-10-12T09:39:23Z
- Publish / stuck / 2026-10-12T09:50:00Z
