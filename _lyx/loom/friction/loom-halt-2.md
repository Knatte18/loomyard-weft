loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/driver-context/.lyx/verify/verify.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 31
trace_file: /home/knatte/Code/loomyard-LYXHUB/driver-context/.lyx/logs/trace-20261010T121556Z-7f5cb705277d1a07-3882892.log

history:
- Preflight / done / 2026-10-10T08:48:57Z
- Loom-Preflight / done / 2026-10-10T08:49:06Z
- Discussion-Write / done / 2026-10-10T09:03:03Z
- Discussion-Bouncer / stuck / 2026-10-10T09:04:02Z
- Discussion-Burler / stuck / 2026-10-10T09:10:26Z
- Discussion-Bouncer / stuck / 2026-10-10T09:11:10Z
- Discussion-Burler / stuck / 2026-10-10T09:16:50Z
- Discussion-Bouncer / stuck / 2026-10-10T09:17:41Z
- Discussion-Burler / stuck / 2026-10-10T09:24:16Z
- Discussion-Bouncer / awaiting / 2026-10-10T09:25:07Z
- Discussion-Bouncer / done / 2026-10-10T09:26:03Z
- Plan-Write / done / 2026-10-10T09:51:08Z
- Plan-Bouncer / stuck / 2026-10-10T09:52:04Z
- Plan-Burler / stuck / 2026-10-10T10:14:08Z
- Plan-Bouncer / stuck / 2026-10-10T10:14:55Z
- Plan-Burler / stuck / 2026-10-10T10:26:53Z
- Plan-Bouncer / stuck / 2026-10-10T10:27:50Z
- Plan-Burler / stuck / 2026-10-10T10:36:32Z
- Plan-Bouncer / awaiting / 2026-10-10T10:37:30Z
- Plan-Bouncer / stuck / 2026-10-10T10:38:22Z
- Plan-Burler / stuck / 2026-10-10T10:45:38Z
- Plan-Bouncer / awaiting / 2026-10-10T10:46:31Z
- Plan-Bouncer / done / 2026-10-10T10:47:25Z
- Batchifier / done / 2026-10-10T10:47:32Z
- Webster / stuck / 2026-10-10T11:59:52Z
- Webster / done / 2026-10-10T12:06:48Z
- Webster-Bouncer / stuck / 2026-10-10T12:07:26Z
- Webster-Burler / stuck / 2026-10-10T12:14:02Z
- Webster-Bouncer / done / 2026-10-10T12:14:39Z
- Describe / done / 2026-10-10T12:15:41Z
- Publish / stuck / 2026-10-10T12:20:43Z
