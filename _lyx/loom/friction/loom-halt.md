loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/pane-inject-safety/.lyx/verify/verify-4.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 19
trace_file: /home/knatte/Code/loomyard-LYXHUB/pane-inject-safety/.lyx/logs/trace-20261011T114322Z-61e34546e70f04a0-1674593.log

history:
- Preflight / done / 2026-10-11T10:15:33Z
- Loom-Preflight / done / 2026-10-11T10:15:39Z
- Discussion-Write / done / 2026-10-11T10:22:04Z
- Discussion-Bouncer / stuck / 2026-10-11T10:22:48Z
- Discussion-Burler / stuck / 2026-10-11T10:30:59Z
- Discussion-Bouncer / stuck / 2026-10-11T10:31:51Z
- Discussion-Burler / stuck / 2026-10-11T10:37:03Z
- Discussion-Bouncer / done / 2026-10-11T10:37:54Z
- Plan-Write / done / 2026-10-11T10:51:44Z
- Plan-Bouncer / stuck / 2026-10-11T10:52:30Z
- Plan-Burler / stuck / 2026-10-11T11:00:20Z
- Plan-Bouncer / done / 2026-10-11T11:01:01Z
- Batchifier / done / 2026-10-11T11:01:11Z
- Webster / done / 2026-10-11T11:34:41Z
- Webster-Bouncer / stuck / 2026-10-11T11:35:20Z
- Webster-Burler / stuck / 2026-10-11T11:41:39Z
- Webster-Bouncer / done / 2026-10-11T11:42:20Z
- Describe / done / 2026-10-11T11:43:12Z
- Publish / stuck / 2026-10-11T11:48:38Z
