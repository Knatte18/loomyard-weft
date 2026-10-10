loom halted blocked at Publish

```
verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/webster-replan-state/.lyx/verify/verify.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 31
trace_file: /home/knatte/Code/loomyard-LYXHUB/webster-replan-state/.lyx/logs/trace-20261010T142857Z-ad5dd94d124b1883-1805532.log

history:
- Preflight / done / 2026-10-10T10:19:06Z
- Loom-Preflight / done / 2026-10-10T10:19:14Z
- Discussion-Write / done / 2026-10-10T10:31:12Z
- Discussion-Bouncer / stuck / 2026-10-10T10:32:03Z
- Discussion-Burler / stuck / 2026-10-10T10:41:32Z
- Discussion-Bouncer / stuck / 2026-10-10T10:42:15Z
- Discussion-Burler / stuck / 2026-10-10T10:55:09Z
- Discussion-Bouncer / stuck / 2026-10-10T10:56:05Z
- Discussion-Burler / stuck / 2026-10-10T11:07:15Z
- Discussion-Bouncer / awaiting / 2026-10-10T11:08:19Z
- Discussion-Bouncer / stuck / 2026-10-10T11:09:36Z
- Discussion-Burler / stuck / 2026-10-10T11:22:42Z
- Discussion-Bouncer / awaiting / 2026-10-10T11:23:48Z
- Discussion-Bouncer / done / 2026-10-10T11:24:32Z
- Plan-Write / done / 2026-10-10T11:44:20Z
- Plan-Bouncer / stuck / 2026-10-10T11:45:04Z
- Plan-Burler / stuck / 2026-10-10T12:06:04Z
- Plan-Bouncer / stuck / 2026-10-10T12:06:56Z
- Plan-Burler / stuck / 2026-10-10T12:26:41Z
- Plan-Bouncer / stuck / 2026-10-10T12:27:46Z
- Plan-Burler / stuck / 2026-10-10T12:42:47Z
- Plan-Bouncer / done / 2026-10-10T12:43:46Z
- Batchifier / done / 2026-10-10T12:44:01Z
- Webster / done / 2026-10-10T13:55:10Z
- Webster-Bouncer / stuck / 2026-10-10T13:55:57Z
- Webster-Burler / stuck / 2026-10-10T14:11:25Z
- Webster-Bouncer / stuck / 2026-10-10T14:12:14Z
- Webster-Burler / stuck / 2026-10-10T14:27:03Z
- Webster-Bouncer / done / 2026-10-10T14:27:46Z
- Describe / done / 2026-10-10T14:28:42Z
- Publish / stuck / 2026-10-10T14:31:33Z
