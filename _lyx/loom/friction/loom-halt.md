loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/run-robustness/.lyx/verify/verify-7.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 34
trace_file: /home/knatte/Code/loomyard-LYXHUB/run-robustness/.lyx/logs/trace-20261012T131809Z-a5076fc1e1c19b3b-2403054.log

history:
- Preflight / done / 2026-10-12T11:23:41Z
- Loom-Preflight / done / 2026-10-12T11:23:59Z
- Discussion-Write / done / 2026-10-12T11:32:25Z
- Discussion-Bouncer / stuck / 2026-10-12T11:33:06Z
- Discussion-Burler-Review / stuck / 2026-10-12T11:36:37Z
- Discussion-Burler-Fix / stuck / 2026-10-12T11:38:07Z
- Discussion-Bouncer / stuck / 2026-10-12T11:38:43Z
- Discussion-Burler-Review / stuck / 2026-10-12T11:41:38Z
- Discussion-Burler-Fix / stuck / 2026-10-12T11:43:32Z
- Discussion-Bouncer / awaiting / 2026-10-12T11:44:14Z
- Discussion-Bouncer / done / 2026-10-12T11:45:28Z
- Plan-Write / done / 2026-10-12T12:04:49Z
- Plan-Bouncer / stuck / 2026-10-12T12:05:12Z
- Plan-Burler-Review / stuck / 2026-10-12T12:16:02Z
- Plan-Burler-Fix / stuck / 2026-10-12T12:18:34Z
- Plan-Bouncer / stuck / 2026-10-12T12:19:04Z
- Plan-Burler-Review / stuck / 2026-10-12T12:25:04Z
- Plan-Burler-Fix / stuck / 2026-10-12T12:26:42Z
- Plan-Bouncer / awaiting / 2026-10-12T12:27:21Z
- Plan-Bouncer / done / 2026-10-12T12:28:29Z
- Batchifier / done / 2026-10-12T12:28:41Z
- Webster / done / 2026-10-12T12:58:03Z
- Webster-Bouncer / stuck / 2026-10-12T12:58:31Z
- Webster-Burler-Review / stuck / 2026-10-12T13:01:32Z
- Webster-Burler-Fix / stuck / 2026-10-12T13:05:09Z
- Webster-Bouncer / done / 2026-10-12T13:05:35Z
- Describe / done / 2026-10-12T13:06:20Z
- Merge-In / done / 2026-10-12T13:11:00Z
- Merge-Bouncer / stuck / 2026-10-12T13:11:34Z
- Merge-Burler-Review / stuck / 2026-10-12T13:13:06Z
- Merge-Burler-Fix / stuck / 2026-10-12T13:16:17Z
- Merge-Bouncer / awaiting / 2026-10-12T13:17:00Z
- Merge-Bouncer / done / 2026-10-12T13:18:07Z
- Publish / stuck / 2026-10-12T13:19:32Z
