loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/runs-overview/.lyx/verify/verify-6.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 34
trace_file: /home/knatte/Code/loomyard-LYXHUB/runs-overview/.lyx/logs/trace-20261012T111519Z-99b00704a5c34a91-466474.log

history:
- Preflight / done / 2026-10-12T08:56:35Z
- Loom-Preflight / done / 2026-10-12T08:56:47Z
- Discussion-Write / done / 2026-10-12T09:05:15Z
- Discussion-Bouncer / stuck / 2026-10-12T09:05:49Z
- Discussion-Burler-Review / stuck / 2026-10-12T09:10:20Z
- Discussion-Burler-Fix / stuck / 2026-10-12T09:12:22Z
- Discussion-Bouncer / stuck / 2026-10-12T09:12:54Z
- Discussion-Burler-Review / stuck / 2026-10-12T09:15:57Z
- Discussion-Burler-Fix / stuck / 2026-10-12T09:17:05Z
- Discussion-Bouncer / done / 2026-10-12T09:17:48Z
- Plan-Write / done / 2026-10-12T09:51:22Z
- Plan-Bouncer / stuck / 2026-10-12T09:52:03Z
- Plan-Burler-Review / stuck / 2026-10-12T10:01:25Z
- Plan-Burler-Fix / stuck / 2026-10-12T10:02:52Z
- Plan-Bouncer / stuck / 2026-10-12T10:03:36Z
- Plan-Burler-Review / stuck / 2026-10-12T10:14:36Z
- Plan-Burler-Fix / stuck / 2026-10-12T10:15:58Z
- Plan-Bouncer / stuck / 2026-10-12T10:16:37Z
- Plan-Burler-Review / stuck / 2026-10-12T10:24:02Z
- Plan-Burler-Fix / stuck / 2026-10-12T10:24:57Z
- Plan-Bouncer / done / 2026-10-12T10:25:29Z
- Batchifier / done / 2026-10-12T10:25:40Z
- Webster / done / 2026-10-12T10:59:48Z
- Webster-Bouncer / stuck / 2026-10-12T11:00:11Z
- Webster-Burler-Review / stuck / 2026-10-12T11:02:45Z
- Webster-Burler-Fix / stuck / 2026-10-12T11:03:11Z
- Webster-Bouncer / done / 2026-10-12T11:03:38Z
- Describe / done / 2026-10-12T11:04:22Z
- Merge-In / done / 2026-10-12T11:10:28Z
- Merge-Bouncer / stuck / 2026-10-12T11:10:56Z
- Merge-Burler-Review / stuck / 2026-10-12T11:13:32Z
- Merge-Burler-Fix / stuck / 2026-10-12T11:14:46Z
- Merge-Bouncer / done / 2026-10-12T11:15:07Z
- Publish / stuck / 2026-10-12T11:16:46Z
