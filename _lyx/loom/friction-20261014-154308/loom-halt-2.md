loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/tender/.lyx/verify/verify-7.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 43
trace_file: /home/knatte/Code/loomyard-LYXHUB/tender/.lyx/logs/trace-20261014T152459Z-b8928c67a8d4866b-278100.log

history:
- Preflight / done / 2026-10-13T14:58:40Z
- Loom-Preflight / done / 2026-10-13T14:58:48Z
- Discussion-Write / done / 2026-10-13T15:07:19Z
- Discussion-Bouncer / stuck / 2026-10-13T15:07:52Z
- Discussion-Burler-Review / stuck / 2026-10-13T15:14:13Z
- Discussion-Burler-Fix / stuck / 2026-10-13T15:15:51Z
- Discussion-Bouncer / stuck / 2026-10-13T15:16:33Z
- Discussion-Burler-Review / stuck / 2026-10-13T15:20:09Z
- Discussion-Burler-Fix / stuck / 2026-10-13T15:22:01Z
- Discussion-Bouncer / stuck / 2026-10-13T15:22:45Z
- Discussion-Burler-Review / stuck / 2026-10-13T15:26:01Z
- Discussion-Burler-Fix / stuck / 2026-10-13T15:28:09Z
- Discussion-Bouncer / awaiting / 2026-10-13T15:28:56Z
- Discussion-Bouncer / stuck / 2026-10-13T15:32:45Z
- Discussion-Burler-Review / stuck / 2026-10-13T15:37:56Z
- Discussion-Burler-Fix / stuck / 2026-10-13T15:39:12Z
- Discussion-Bouncer / awaiting / 2026-10-13T15:39:59Z
- Discussion-Bouncer / done / 2026-10-13T15:41:17Z
- Plan-Write / done / 2026-10-13T15:59:18Z
- Plan-Bouncer / stuck / 2026-10-14T14:17:22Z
- Plan-Burler-Review / stuck / 2026-10-14T14:26:02Z
- Plan-Burler-Fix / stuck / 2026-10-14T14:28:07Z
- Plan-Bouncer / stuck / 2026-10-14T14:28:46Z
- Plan-Burler-Review / stuck / 2026-10-14T14:35:42Z
- Plan-Burler-Fix / stuck / 2026-10-14T14:37:55Z
- Plan-Bouncer / stuck / 2026-10-14T14:38:37Z
- Plan-Burler-Review / stuck / 2026-10-14T14:43:14Z
- Plan-Burler-Fix / stuck / 2026-10-14T14:44:22Z
- Plan-Bouncer / done / 2026-10-14T14:45:07Z
- Batchifier / done / 2026-10-14T14:45:18Z
- Webster / stuck / 2026-10-14T15:07:50Z
- Webster / done / 2026-10-14T15:15:20Z
- Webster-Bouncer / stuck / 2026-10-14T15:15:49Z
- Webster-Burler-Review / stuck / 2026-10-14T15:18:03Z
- Webster-Burler-Fix / stuck / 2026-10-14T15:19:10Z
- Webster-Bouncer / done / 2026-10-14T15:19:40Z
- Describe / done / 2026-10-14T15:20:39Z
- Merge-In / done / 2026-10-14T15:22:39Z
- Merge-Bouncer / stuck / 2026-10-14T15:23:14Z
- Merge-Burler-Review / stuck / 2026-10-14T15:24:02Z
- Merge-Burler-Fix / stuck / 2026-10-14T15:24:26Z
- Merge-Bouncer / done / 2026-10-14T15:24:48Z
- Publish / stuck / 2026-10-14T15:27:04Z
