loom halted blocked at Webster

```
batch 01-paced-text-submit: record-batch refused with plan_drifted: webster: on-disk plan fingerprint does not match this run's recorded state: on-disk plan fingerprint 93affdee43ffa9461f0399b86b55d281d7617391459266b03c12dd77275e0dc8 does not match this run's recorded fingerprint 83f0527d24b6e8dfa9dd5ab6b476a2aca7ff21fcbb151d60ccec2983c77e22a3; the plan changed since state.json was created; way forward: run `lyx webster rebaseline --card 01` to accept the edit, or restore the plan the run recorded with "lyx webster restore-plan", otherwise 1) lyx webster reset --to start; 2) lyx webster run --fresh
```

producer: Webster
state: blocked
anomaly: escalation-to-human
history_entries: 26
trace_file: /home/knatte/Code/loomyard-LYXHUB/bugfix-sessions/.lyx/logs/trace-20261007T052818Z-21d8305b80742af6-1794715.log

history:
- Preflight / done / 2026-10-06T10:19:54Z
- Loom-Preflight / done / 2026-10-06T10:19:59Z
- Discussion-Write / done / 2026-10-06T10:25:35Z
- Discussion-Bouncer / stuck / 2026-10-06T10:26:16Z
- Discussion-Burler / stuck / 2026-10-06T10:29:00Z
- Discussion-Bouncer / stuck / 2026-10-06T10:29:47Z
- Discussion-Burler / stuck / 2026-10-06T10:44:48Z
- Discussion-Bouncer / stuck / 2026-10-06T10:45:30Z
- Discussion-Burler / stuck / 2026-10-06T10:48:18Z
- Discussion-Bouncer / awaiting / 2026-10-06T10:49:06Z
- Discussion-Bouncer / done / 2026-10-06T10:49:51Z
- Plan-Write / stuck / 2026-10-06T10:50:41Z
- Plan-Write / done / 2026-10-06T11:09:25Z
- Plan-Bouncer / stuck / 2026-10-06T11:10:36Z
- Plan-Burler / stuck / 2026-10-06T11:25:13Z
- Plan-Bouncer / stuck / 2026-10-06T11:25:47Z
- Plan-Burler / stuck / 2026-10-06T11:29:40Z
- Plan-Bouncer / done / 2026-10-06T11:30:15Z
- Batchifier / done / 2026-10-06T11:30:22Z
- Plan-Write / goto / 2026-10-07T05:12:17Z
- Plan-Write / done / 2026-10-07T05:23:24Z
- Plan-Bouncer / stuck / 2026-10-07T05:23:52Z
- Plan-Burler / stuck / 2026-10-07T05:27:38Z
- Plan-Bouncer / done / 2026-10-07T05:28:06Z
- Batchifier / done / 2026-10-07T05:28:12Z
- Webster / stuck / 2026-10-07T05:32:01Z
