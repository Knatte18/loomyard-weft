loom halted blocked at Publish

```
verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/reed-hygiene/.lyx/verify/verify-3.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 23
trace_file: /home/knatte/Code/loomyard-LYXHUB/reed-hygiene/.lyx/logs/trace-20261010T165437Z-1d7e56386c0611d6-4091123.log

history:
- Preflight / done / 2026-10-10T15:09:17Z
- Loom-Preflight / done / 2026-10-10T15:09:20Z
- Discussion-Write / done / 2026-10-10T15:19:30Z
- Discussion-Bouncer / stuck / 2026-10-10T15:20:19Z
- Discussion-Burler / stuck / 2026-10-10T15:27:18Z
- Discussion-Bouncer / stuck / 2026-10-10T15:28:05Z
- Discussion-Burler / stuck / 2026-10-10T15:34:06Z
- Discussion-Bouncer / done / 2026-10-10T15:34:56Z
- Plan-Write / done / 2026-10-10T15:47:18Z
- Plan-Bouncer / stuck / 2026-10-10T15:48:05Z
- Plan-Burler / stuck / 2026-10-10T15:58:29Z
- Plan-Bouncer / stuck / 2026-10-10T15:59:16Z
- Plan-Burler / stuck / 2026-10-10T16:05:37Z
- Plan-Bouncer / done / 2026-10-10T16:06:20Z
- Batchifier / done / 2026-10-10T16:06:30Z
- Webster / done / 2026-10-10T16:41:54Z
- Webster-Bouncer / stuck / 2026-10-10T16:42:34Z
- Webster-Burler / stuck / 2026-10-10T16:48:52Z
- Webster-Bouncer / stuck / 2026-10-10T16:49:36Z
- Webster-Burler / stuck / 2026-10-10T16:53:00Z
- Webster-Bouncer / done / 2026-10-10T16:53:33Z
- Describe / done / 2026-10-10T16:54:29Z
- Publish / stuck / 2026-10-10T16:55:19Z
