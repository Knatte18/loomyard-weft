loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/reed-multi-window/.lyx/verify/verify-6.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 38
trace_file: /home/knatte/Code/loomyard-LYXHUB/reed-multi-window/.lyx/logs/trace-20261013T064950Z-27c0e420b6f34013-2178583.log

history:
- Preflight / done / 2026-10-12T17:05:06Z
- Loom-Preflight / done / 2026-10-12T17:05:17Z
- Discussion-Write / done / 2026-10-12T17:14:05Z
- Discussion-Bouncer / stuck / 2026-10-12T17:14:43Z
- Discussion-Burler-Review / stuck / 2026-10-12T17:18:37Z
- Discussion-Burler-Fix / stuck / 2026-10-12T17:20:15Z
- Discussion-Bouncer / stuck / 2026-10-12T17:20:59Z
- Discussion-Burler-Review / stuck / 2026-10-12T17:23:52Z
- Discussion-Burler-Fix / stuck / 2026-10-12T17:25:57Z
- Discussion-Bouncer / awaiting / 2026-10-12T17:26:37Z
- Discussion-Bouncer / done / 2026-10-12T17:27:50Z
- Plan-Write / done / 2026-10-12T18:02:08Z
- Plan-Bouncer / stuck / 2026-10-12T18:02:49Z
- Plan-Burler-Review / stuck / 2026-10-12T18:17:42Z
- Plan-Burler-Fix / stuck / 2026-10-12T18:21:27Z
- Plan-Bouncer / stuck / 2026-10-12T18:22:05Z
- Plan-Burler-Review / stuck / 2026-10-12T18:25:24Z
- Plan-Burler-Fix / stuck / 2026-10-12T18:26:55Z
- Plan-Bouncer / done / 2026-10-12T18:27:30Z
- Batchifier / done / 2026-10-12T18:27:41Z
- Webster / done / 2026-10-12T18:55:12Z
- Webster-Bouncer / stuck / 2026-10-12T18:55:42Z
- Webster-Burler-Review / stuck / 2026-10-12T18:58:21Z
- Webster-Burler-Fix / stuck / 2026-10-12T19:01:16Z
- Webster-Bouncer / done / 2026-10-12T19:01:41Z
- Describe / done / 2026-10-12T19:02:23Z
- Merge-In / stuck / 2026-10-12T19:03:53Z
- Webster-Burler-Review / goto / 2026-10-12T19:04:25Z
- Webster-Burler-Review / stuck / 2026-10-13T06:43:17Z
- Webster-Burler-Fix / stuck / 2026-10-13T06:45:15Z
- Webster-Bouncer / stuck / 2026-10-13T06:45:48Z
- Webster-Burler-Review / stuck / 2026-10-13T06:46:29Z
- Webster-Burler-Fix / stuck / 2026-10-13T06:46:44Z
- Webster-Bouncer / done / 2026-10-13T06:47:09Z
- Describe / done / 2026-10-13T06:47:50Z
- Merge-In / done / 2026-10-13T06:49:42Z
- Merge-Bouncer / done / 2026-10-13T06:49:46Z
- Publish / stuck / 2026-10-13T06:51:14Z
