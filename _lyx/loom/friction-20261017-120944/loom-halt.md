loom halted blocked at Merge-In

```
verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/gate-wait-cache-cap/.lyx/verify/verify-3.log; way forward: run "lyx loom goto --to Darn-Write", then "lyx loom resume", in the task worktree; the Darn-Write spawn renders the Publish failure record and its gate runs the failing tests
```

producer: Merge-In
state: blocked
anomaly: escalation-to-human
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYXHUB/gate-wait-cache-cap/.lyx/logs/trace-20261017T113906Z-2e2d25923bfebfc3-4103304.log

history:
- Preflight / done / 2026-10-17T11:22:54Z
- Darn-Write / done / 2026-10-17T11:32:13Z
- Darn-Bouncer / stuck / 2026-10-17T11:32:36Z
- Darn-Burler-Review / stuck / 2026-10-17T11:34:25Z
- Darn-Burler-Fix / stuck / 2026-10-17T11:37:50Z
- Darn-Bouncer / done / 2026-10-17T11:38:20Z
- Describe / done / 2026-10-17T11:38:55Z
- Merge-In / stuck / 2026-10-17T11:41:28Z
