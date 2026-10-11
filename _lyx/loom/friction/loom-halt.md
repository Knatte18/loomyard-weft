loom halted blocked at Publish

```
verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/orch-wake-policy/.lyx/verify/verify-6.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 25
trace_file: /home/knatte/Code/loomyard-LYXHUB/orch-wake-policy/.lyx/logs/trace-20261011T182050Z-08a0675039bf24cd-2221868.log

history:
- Preflight / done / 2026-10-11T16:18:32Z
- Loom-Preflight / done / 2026-10-11T16:18:42Z
- Discussion-Write / done / 2026-10-11T16:28:51Z
- Discussion-Bouncer / stuck / 2026-10-11T16:29:24Z
- Discussion-Burler / stuck / 2026-10-11T16:35:23Z
- Discussion-Bouncer / stuck / 2026-10-11T16:35:53Z
- Discussion-Burler / stuck / 2026-10-11T16:43:54Z
- Discussion-Bouncer / stuck / 2026-10-11T16:44:38Z
- Discussion-Burler / stuck / 2026-10-11T16:49:25Z
- Discussion-Bouncer / done / 2026-10-11T16:49:56Z
- Plan-Write / done / 2026-10-11T17:07:04Z
- Plan-Bouncer / stuck / 2026-10-11T17:07:42Z
- Plan-Burler / stuck / 2026-10-11T17:21:56Z
- Plan-Bouncer / stuck / 2026-10-11T17:22:33Z
- Plan-Burler / stuck / 2026-10-11T17:32:09Z
- Plan-Bouncer / done / 2026-10-11T17:32:48Z
- Batchifier / done / 2026-10-11T17:32:58Z
- Webster / done / 2026-10-11T18:11:14Z
- Webster-Bouncer / stuck / 2026-10-11T18:11:35Z
- Webster-Burler / stuck / 2026-10-11T18:17:11Z
- Webster-Bouncer / stuck / 2026-10-11T18:17:37Z
- Webster-Burler / stuck / 2026-10-11T18:19:42Z
- Webster-Bouncer / done / 2026-10-11T18:20:08Z
- Describe / done / 2026-10-11T18:20:48Z
- Publish / stuck / 2026-10-11T18:20:54Z
