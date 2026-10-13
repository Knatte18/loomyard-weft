loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/webster-pack-last/.lyx/verify/verify-5.log; way forward: run "lyx loom goto --to Darn-Write", then "lyx loom resume", in the task worktree; the Darn-Write spawn renders the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 12
trace_file: /home/knatte/Code/loomyard-LYXHUB/webster-pack-last/.lyx/logs/trace-20261013T140746Z-8209e15ce9626307-2319764.log

history:
- Preflight / done / 2026-10-13T13:37:02Z
- Darn-Write / done / 2026-10-13T13:50:19Z
- Darn-Bouncer / stuck / 2026-10-13T13:53:55Z
- Darn-Burler-Review / stuck / 2026-10-13T13:55:28Z
- Darn-Burler-Fix / stuck / 2026-10-13T14:00:21Z
- Darn-Bouncer / stuck / 2026-10-13T14:01:01Z
- Darn-Burler-Review / stuck / 2026-10-13T14:02:06Z
- Darn-Burler-Fix / stuck / 2026-10-13T14:04:38Z
- Darn-Bouncer / done / 2026-10-13T14:05:05Z
- Describe / done / 2026-10-13T14:05:38Z
- Merge-In / done / 2026-10-13T14:07:42Z
- Publish / stuck / 2026-10-13T14:09:33Z
