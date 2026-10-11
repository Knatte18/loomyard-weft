loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/shuttle-hook-first/.lyx/verify/verify-4.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 24
trace_file: /home/knatte/Code/loomyard-LYXHUB/shuttle-hook-first/.lyx/logs/trace-20261011T172347Z-697bda1327618e1e-1362533.log

history:
- Preflight / done / 2026-10-11T14:29:09Z
- Loom-Preflight / done / 2026-10-11T14:29:13Z
- Discussion-Write / done / 2026-10-11T14:38:55Z
- Discussion-Bouncer / stuck / 2026-10-11T14:39:30Z
- Discussion-Burler / stuck / 2026-10-11T14:46:21Z
- Discussion-Bouncer / stuck / 2026-10-11T14:46:48Z
- Discussion-Burler / stuck / 2026-10-11T14:52:34Z
- Discussion-Bouncer / stuck / 2026-10-11T14:53:16Z
- Discussion-Burler / stuck / 2026-10-11T14:59:06Z
- Discussion-Bouncer / awaiting / 2026-10-11T14:59:52Z
- Discussion-Bouncer / done / 2026-10-11T15:00:48Z
- Plan-Write / done / 2026-10-11T15:10:13Z
- Plan-Bouncer / stuck / 2026-10-11T15:11:05Z
- Plan-Burler / stuck / 2026-10-11T15:20:47Z
- Plan-Bouncer / done / 2026-10-11T15:21:17Z
- Batchifier / done / 2026-10-11T15:21:28Z
- Webster / done / 2026-10-11T17:12:07Z
- Webster-Bouncer / stuck / 2026-10-11T17:12:38Z
- Webster-Burler / stuck / 2026-10-11T17:20:27Z
- Webster-Bouncer / stuck / 2026-10-11T17:20:51Z
- Webster-Burler / stuck / 2026-10-11T17:22:30Z
- Webster-Bouncer / done / 2026-10-11T17:22:55Z
- Describe / done / 2026-10-11T17:23:36Z
- Publish / stuck / 2026-10-11T17:27:54Z
