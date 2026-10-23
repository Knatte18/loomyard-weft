loom halted blocked at Publish

```
publish gate failed after merging parent branch "main" (exit code 1); output: /home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYXHUB/orch-compact-repeats/.lyx/verify/verify-4.log; way forward: run "lyx loom goto --to Darn-Write", then "lyx loom resume", in the task worktree; the Darn-Write spawn renders the Publish failure record and its gate runs the failing tests
```

producer: Publish
state: blocked
anomaly: escalation-to-human
history_entries: 9
trace_file: /home/AD.NORCERESEARCH.NO/hanf/Code/Knatte18/loomyard-LYXHUB/orch-compact-repeats/.lyx/logs/trace-20261023T071019Z-daf63119c8fd23bb-1669390.log

history:
- Preflight / done / 2026-10-23T06:48:07Z
- Darn-Write / done / 2026-10-23T07:01:27Z
- Darn-Bouncer / stuck / 2026-10-23T07:01:43Z
- Darn-Burler-Review / stuck / 2026-10-23T07:03:30Z
- Darn-Burler-Fix / stuck / 2026-10-23T07:07:21Z
- Darn-Bouncer / done / 2026-10-23T07:07:40Z
- Describe / done / 2026-10-23T07:08:07Z
- Merge-In / done / 2026-10-23T07:10:19Z
- Publish / stuck / 2026-10-23T07:13:16Z
