loom halted blocked at Webster

```
run-exit audit found 1 correctness finding(s): 1) parent-write: Master wrote "/home/knatte/Code/loomyard-LYXHUB/driver-context/internal/stencilcli/cli_integration_test.go" — Master may write only its two contract files (outcome.yaml and summary.md); any other write means Master implemented a batch itself or hand-wrote a batch report; way forward: 1) git checkout 0e9431f49a27ae425f1183aa2ce0c0cc163e66e7 -- /home/knatte/Code/loomyard-LYXHUB/driver-context/internal/stencilcli/cli_integration_test.go (rm a path the head does not hold); 2) lyx webster accept-audit; 3) re-step the Webster row
```

producer: Webster
state: blocked
anomaly: escalation-to-human
history_entries: 25
trace_file: /home/knatte/Code/loomyard-LYXHUB/driver-context/.lyx/logs/trace-20261010T104738Z-53eca17d288139a0-1232417.log

history:
- Preflight / done / 2026-10-10T08:48:57Z
- Loom-Preflight / done / 2026-10-10T08:49:06Z
- Discussion-Write / done / 2026-10-10T09:03:03Z
- Discussion-Bouncer / stuck / 2026-10-10T09:04:02Z
- Discussion-Burler / stuck / 2026-10-10T09:10:26Z
- Discussion-Bouncer / stuck / 2026-10-10T09:11:10Z
- Discussion-Burler / stuck / 2026-10-10T09:16:50Z
- Discussion-Bouncer / stuck / 2026-10-10T09:17:41Z
- Discussion-Burler / stuck / 2026-10-10T09:24:16Z
- Discussion-Bouncer / awaiting / 2026-10-10T09:25:07Z
- Discussion-Bouncer / done / 2026-10-10T09:26:03Z
- Plan-Write / done / 2026-10-10T09:51:08Z
- Plan-Bouncer / stuck / 2026-10-10T09:52:04Z
- Plan-Burler / stuck / 2026-10-10T10:14:08Z
- Plan-Bouncer / stuck / 2026-10-10T10:14:55Z
- Plan-Burler / stuck / 2026-10-10T10:26:53Z
- Plan-Bouncer / stuck / 2026-10-10T10:27:50Z
- Plan-Burler / stuck / 2026-10-10T10:36:32Z
- Plan-Bouncer / awaiting / 2026-10-10T10:37:30Z
- Plan-Bouncer / stuck / 2026-10-10T10:38:22Z
- Plan-Burler / stuck / 2026-10-10T10:45:38Z
- Plan-Bouncer / awaiting / 2026-10-10T10:46:31Z
- Plan-Bouncer / done / 2026-10-10T10:47:25Z
- Batchifier / done / 2026-10-10T10:47:32Z
- Webster / stuck / 2026-10-10T11:59:52Z
