loom halted blocked at Publish

```
verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/webster-audit-gaps/.lyx/verify/verify.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 26
trace_file: /home/knatte/Code/loomyard-LYXHUB/webster-audit-gaps/.lyx/logs/trace-20261010T163910Z-2476b783608e97b3-3757127.log

history:
- Preflight / done / 2026-10-10T15:09:18Z
- Loom-Preflight / done / 2026-10-10T15:09:30Z
- Discussion-Write / done / 2026-10-10T15:19:50Z
- Discussion-Bouncer / stuck / 2026-10-10T15:20:36Z
- Discussion-Burler / stuck / 2026-10-10T15:27:08Z
- Discussion-Bouncer / stuck / 2026-10-10T15:27:56Z
- Discussion-Burler / stuck / 2026-10-10T15:34:23Z
- Discussion-Bouncer / stuck / 2026-10-10T15:35:07Z
- Discussion-Burler / stuck / 2026-10-10T15:42:27Z
- Discussion-Bouncer / awaiting / 2026-10-10T15:43:14Z
- Discussion-Bouncer / done / 2026-10-10T15:44:22Z
- Plan-Write / done / 2026-10-10T15:56:30Z
- Plan-Bouncer / stuck / 2026-10-10T15:57:13Z
- Plan-Burler / stuck / 2026-10-10T16:04:12Z
- Plan-Bouncer / stuck / 2026-10-10T16:04:56Z
- Plan-Burler / stuck / 2026-10-10T16:11:10Z
- Plan-Bouncer / done / 2026-10-10T16:11:55Z
- Batchifier / done / 2026-10-10T16:12:06Z
- Webster / done / 2026-10-10T16:28:24Z
- Webster-Bouncer / stuck / 2026-10-10T16:29:03Z
- Webster-Burler / stuck / 2026-10-10T16:35:10Z
- Webster-Bouncer / stuck / 2026-10-10T16:35:44Z
- Webster-Burler / stuck / 2026-10-10T16:37:28Z
- Webster-Bouncer / done / 2026-10-10T16:38:08Z
- Describe / done / 2026-10-10T16:39:02Z
- Publish / stuck / 2026-10-10T16:39:42Z
