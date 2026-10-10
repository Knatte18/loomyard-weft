loom halted blocked at Publish

```
verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/discussion-multi-llm/.lyx/verify/verify.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 27
trace_file: /home/knatte/Code/loomyard-LYXHUB/discussion-multi-llm/.lyx/logs/trace-20261010T104339Z-e2a1c9d600ca820d-1119760.log

history:
- Preflight / done / 2026-10-10T08:44:19Z
- Loom-Preflight / done / 2026-10-10T08:44:28Z
- Discussion-Write / done / 2026-10-10T08:54:27Z
- Discussion-Bouncer / stuck / 2026-10-10T08:55:12Z
- Discussion-Burler / stuck / 2026-10-10T09:00:41Z
- Discussion-Bouncer / stuck / 2026-10-10T09:01:27Z
- Discussion-Burler / stuck / 2026-10-10T09:06:19Z
- Discussion-Bouncer / done / 2026-10-10T09:07:03Z
- Plan-Write / done / 2026-10-10T09:23:17Z
- Plan-Bouncer / stuck / 2026-10-10T09:25:40Z
- Plan-Burler / stuck / 2026-10-10T09:38:11Z
- Plan-Write / goto / 2026-10-10T09:38:43Z
- Plan-Write / done / 2026-10-10T09:53:43Z
- Plan-Bouncer / stuck / 2026-10-10T09:54:26Z
- Plan-Burler / stuck / 2026-10-10T10:06:33Z
- Plan-Bouncer / stuck / 2026-10-10T10:07:18Z
- Plan-Burler / stuck / 2026-10-10T10:12:00Z
- Plan-Bouncer / done / 2026-10-10T10:12:44Z
- Batchifier / done / 2026-10-10T10:12:53Z
- Webster / done / 2026-10-10T10:29:28Z
- Webster-Bouncer / stuck / 2026-10-10T10:30:09Z
- Webster-Burler / stuck / 2026-10-10T10:36:20Z
- Webster-Bouncer / stuck / 2026-10-10T10:37:00Z
- Webster-Burler / stuck / 2026-10-10T10:41:58Z
- Webster-Bouncer / done / 2026-10-10T10:42:36Z
- Describe / done / 2026-10-10T10:43:32Z
- Publish / stuck / 2026-10-10T10:45:02Z
