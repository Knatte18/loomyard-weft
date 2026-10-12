loom halted blocked at Merge-In

```
verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/operator-followups/.lyx/verify/verify-3.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Merge-In
state: blocked
anomaly: escalation-to-human
history_entries: 24
trace_file: /home/knatte/Code/loomyard-LYXHUB/operator-followups/.lyx/logs/trace-20261012T144415Z-a1a080e1b2a50b05-176883.log

history:
- Preflight / done / 2026-10-12T13:44:37Z
- Loom-Preflight / done / 2026-10-12T13:44:49Z
- Discussion-Write / done / 2026-10-12T13:55:08Z
- Discussion-Bouncer / stuck / 2026-10-12T13:55:37Z
- Discussion-Burler-Review / stuck / 2026-10-12T13:59:57Z
- Discussion-Burler-Fix / stuck / 2026-10-12T14:01:09Z
- Discussion-Bouncer / stuck / 2026-10-12T14:01:48Z
- Discussion-Burler-Review / stuck / 2026-10-12T14:04:07Z
- Discussion-Burler-Fix / stuck / 2026-10-12T14:05:14Z
- Discussion-Bouncer / awaiting / 2026-10-12T14:05:48Z
- Discussion-Bouncer / done / 2026-10-12T14:06:55Z
- Plan-Write / done / 2026-10-12T14:20:18Z
- Plan-Bouncer / stuck / 2026-10-12T14:20:54Z
- Plan-Burler-Review / stuck / 2026-10-12T14:28:37Z
- Plan-Burler-Fix / stuck / 2026-10-12T14:29:35Z
- Plan-Bouncer / done / 2026-10-12T14:30:07Z
- Batchifier / done / 2026-10-12T14:30:11Z
- Webster / done / 2026-10-12T14:39:49Z
- Webster-Bouncer / stuck / 2026-10-12T14:40:21Z
- Webster-Burler-Review / stuck / 2026-10-12T14:42:02Z
- Webster-Burler-Fix / stuck / 2026-10-12T14:42:58Z
- Webster-Bouncer / done / 2026-10-12T14:43:21Z
- Describe / done / 2026-10-12T14:44:04Z
- Merge-In / stuck / 2026-10-12T14:44:47Z
