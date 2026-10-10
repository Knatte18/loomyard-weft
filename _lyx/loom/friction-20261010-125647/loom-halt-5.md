loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/test-suite-weight/.lyx/verify/verify.log; way forward: run "lyx loom goto --to Webster-Burler", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 34
trace_file: /home/knatte/Code/loomyard-LYXHUB/test-suite-weight/.lyx/logs/trace-20261010T115234Z-f8c513a318d5ae6b-3303581.log

history:
- Preflight / done / 2026-10-10T07:16:36Z
- Loom-Preflight / done / 2026-10-10T07:16:42Z
- Discussion-Write / done / 2026-10-10T07:31:32Z
- Discussion-Bouncer / stuck / 2026-10-10T07:32:13Z
- Discussion-Burler / stuck / 2026-10-10T07:40:51Z
- Discussion-Bouncer / stuck / 2026-10-10T07:41:37Z
- Discussion-Burler / stuck / 2026-10-10T07:47:22Z
- Discussion-Bouncer / stuck / 2026-10-10T07:48:08Z
- Discussion-Burler / stuck / 2026-10-10T07:52:30Z
- Discussion-Bouncer / awaiting / 2026-10-10T07:53:15Z
- Discussion-Bouncer / done / 2026-10-10T07:54:22Z
- Plan-Write / done / 2026-10-10T08:22:48Z
- Plan-Bouncer / stuck / 2026-10-10T08:25:21Z
- Plan-Burler / stuck / 2026-10-10T08:40:08Z
- Plan-Bouncer / stuck / 2026-10-10T08:40:48Z
- Plan-Burler / stuck / 2026-10-10T08:54:36Z
- Plan-Bouncer / stuck / 2026-10-10T08:55:26Z
- Plan-Burler / stuck / 2026-10-10T09:08:38Z
- Plan-Bouncer / awaiting / 2026-10-10T09:09:31Z
- Plan-Bouncer / done / 2026-10-10T09:10:13Z
- Batchifier / done / 2026-10-10T09:10:19Z
- Webster / stuck / 2026-10-10T09:31:55Z
- Webster / stuck / 2026-10-10T09:35:13Z
- Webster / goto / 2026-10-10T09:35:34Z
- Webster / stuck / 2026-10-10T09:36:10Z
- Webster / stuck / 2026-10-10T10:10:00Z
- Webster / done / 2026-10-10T11:05:27Z
- Webster-Bouncer / stuck / 2026-10-10T11:06:17Z
- Webster-Burler / stuck / 2026-10-10T11:41:22Z
- Webster-Bouncer / stuck / 2026-10-10T11:42:12Z
- Webster-Burler / stuck / 2026-10-10T11:50:30Z
- Webster-Bouncer / done / 2026-10-10T11:51:12Z
- Describe / done / 2026-10-10T11:52:27Z
- Publish / stuck / 2026-10-10T12:00:01Z
