loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/stencilstore-dev-warning-noise/.lyx/verify/verify-3.log; way forward: run "lyx loom goto --to Darn-Write", then "lyx loom resume", in the task worktree; the Darn-Write spawn renders the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 9
trace_file: /home/knatte/Code/loomyard-LYXHUB/stencilstore-dev-warning-noise/.lyx/logs/trace-20261017T114732Z-9febcbd9e6ce9029-401497.log

history:
- Preflight / done / 2026-10-17T11:31:30Z
- Darn-Write / done / 2026-10-17T11:41:32Z
- Darn-Bouncer / stuck / 2026-10-17T11:41:53Z
- Darn-Burler-Review / stuck / 2026-10-17T11:43:20Z
- Darn-Burler-Fix / stuck / 2026-10-17T11:43:36Z
- Darn-Bouncer / done / 2026-10-17T11:44:07Z
- Describe / done / 2026-10-17T11:44:36Z
- Merge-In / done / 2026-10-17T11:47:28Z
- Publish / stuck / 2026-10-17T11:49:04Z
