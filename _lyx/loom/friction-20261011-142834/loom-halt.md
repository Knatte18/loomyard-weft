loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/shuttle-startup-ready/.lyx/verify/verify-4.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 22
trace_file: /home/knatte/Code/loomyard-LYXHUB/shuttle-startup-ready/.lyx/logs/trace-20261011T140841Z-97aa35a6d9a30eba-3728739.log

history:
- Preflight / done / 2026-10-11T13:11:25Z
- Loom-Preflight / done / 2026-10-11T13:11:28Z
- Discussion-Write / done / 2026-10-11T13:17:13Z
- Discussion-Bouncer / stuck / 2026-10-11T13:17:54Z
- Discussion-Burler / stuck / 2026-10-11T13:22:53Z
- Discussion-Bouncer / stuck / 2026-10-11T13:23:30Z
- Discussion-Burler / stuck / 2026-10-11T13:28:30Z
- Discussion-Bouncer / awaiting / 2026-10-11T13:29:12Z
- Discussion-Bouncer / done / 2026-10-11T13:30:16Z
- Plan-Write / done / 2026-10-11T13:36:44Z
- Plan-Bouncer / stuck / 2026-10-11T13:37:29Z
- Plan-Burler / stuck / 2026-10-11T13:45:27Z
- Plan-Bouncer / stuck / 2026-10-11T13:46:09Z
- Plan-Burler / stuck / 2026-10-11T13:53:35Z
- Plan-Bouncer / done / 2026-10-11T13:54:10Z
- Batchifier / done / 2026-10-11T13:54:21Z
- Webster / done / 2026-10-11T14:04:30Z
- Webster-Bouncer / stuck / 2026-10-11T14:05:07Z
- Webster-Burler / stuck / 2026-10-11T14:07:08Z
- Webster-Bouncer / done / 2026-10-11T14:07:44Z
- Describe / done / 2026-10-11T14:08:31Z
- Publish / stuck / 2026-10-11T14:11:33Z
