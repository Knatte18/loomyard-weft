loom halted blocked at Publish

```
publish_verify failed after merging parent branch "main" (exit code 1); output: /home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYXHUB/arbiter-verdict-quoting/.lyx/verify/verify-4.log; way forward: run "lyx loom goto --to Darn-Write", then "lyx loom resume", in the task worktree; the Darn-Write spawn renders the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 9
trace_file: /home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYXHUB/arbiter-verdict-quoting/.lyx/logs/trace-20261020T062836Z-8c1f2c0500baf0ee-737110.log

history:
- Preflight / done / 2026-10-20T06:02:03Z
- Darn-Write / done / 2026-10-20T06:14:31Z
- Darn-Bouncer / stuck / 2026-10-20T06:14:49Z
- Darn-Burler-Review / stuck / 2026-10-20T06:16:08Z
- Darn-Burler-Fix / stuck / 2026-10-20T06:27:52Z
- Darn-Bouncer / done / 2026-10-20T06:28:12Z
- Describe / done / 2026-10-20T06:28:35Z
- Merge-In / done / 2026-10-20T06:28:36Z
- Publish / stuck / 2026-10-20T06:34:31Z
