loom halted blocked at Webster

```
batch 07-proc-process-listing: record-batch and then recover-batch both refused with batch_failed; the done-check says plan handles for cards 8-10 (internal/pausecli#resumeCmd, #pauseCmd, #statusCmd, internal/pauseengine#SessionStatus, #Revive and others) still do not resolve, and the recovery ended with 'way forward: lyx webster recover-batch 07'
```

producer: Webster
state: blocked
anomaly: escalation-to-human
history_entries: 31
trace_file: /home/knatte/Code/loomyard-LYXHUB/tender/.lyx/logs/trace-20261014T144530Z-d80d08ba07f74fdf-3765229.log

history:
- Preflight / done / 2026-10-13T14:58:40Z
- Loom-Preflight / done / 2026-10-13T14:58:48Z
- Discussion-Write / done / 2026-10-13T15:07:19Z
- Discussion-Bouncer / stuck / 2026-10-13T15:07:52Z
- Discussion-Burler-Review / stuck / 2026-10-13T15:14:13Z
- Discussion-Burler-Fix / stuck / 2026-10-13T15:15:51Z
- Discussion-Bouncer / stuck / 2026-10-13T15:16:33Z
- Discussion-Burler-Review / stuck / 2026-10-13T15:20:09Z
- Discussion-Burler-Fix / stuck / 2026-10-13T15:22:01Z
- Discussion-Bouncer / stuck / 2026-10-13T15:22:45Z
- Discussion-Burler-Review / stuck / 2026-10-13T15:26:01Z
- Discussion-Burler-Fix / stuck / 2026-10-13T15:28:09Z
- Discussion-Bouncer / awaiting / 2026-10-13T15:28:56Z
- Discussion-Bouncer / stuck / 2026-10-13T15:32:45Z
- Discussion-Burler-Review / stuck / 2026-10-13T15:37:56Z
- Discussion-Burler-Fix / stuck / 2026-10-13T15:39:12Z
- Discussion-Bouncer / awaiting / 2026-10-13T15:39:59Z
- Discussion-Bouncer / done / 2026-10-13T15:41:17Z
- Plan-Write / done / 2026-10-13T15:59:18Z
- Plan-Bouncer / stuck / 2026-10-14T14:17:22Z
- Plan-Burler-Review / stuck / 2026-10-14T14:26:02Z
- Plan-Burler-Fix / stuck / 2026-10-14T14:28:07Z
- Plan-Bouncer / stuck / 2026-10-14T14:28:46Z
- Plan-Burler-Review / stuck / 2026-10-14T14:35:42Z
- Plan-Burler-Fix / stuck / 2026-10-14T14:37:55Z
- Plan-Bouncer / stuck / 2026-10-14T14:38:37Z
- Plan-Burler-Review / stuck / 2026-10-14T14:43:14Z
- Plan-Burler-Fix / stuck / 2026-10-14T14:44:22Z
- Plan-Bouncer / done / 2026-10-14T14:45:07Z
- Batchifier / done / 2026-10-14T14:45:18Z
- Webster / stuck / 2026-10-14T15:07:50Z
