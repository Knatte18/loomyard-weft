loom halted failed at Friction-Reflect

```
loop-exited: loop 6350aefe7259f99e ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/shed/darn-writer-anatomy/steps/loop.log; step Friction-Reflect was in flight under trace 4b42ba2d5e2c5b10; loop exit: not recorded; loop log: {"continue":true,"envelope_path":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/shed/darn-writer-anatomy/steps/f64a562d506b9430.json","friction":"skipped","history_length":8,"next":"Drift-Recheck","ok":true,"outcome":"done","output":"","producer":"Code-Bouncer","progress":{"name":"Drift-Recheck","step":9,"steps":13},"reason":"","run_id":"darn-writer-anatomy","state":"running","trace_file":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/logs/trace-20261108T144129Z-f64a562d506b9430-373202.log"} | {"continue":true,"envelope_path":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/shed/darn-writer-anatomy/steps/9147c44ecc9fe089.json","friction":"skipped","history_length":11,"next":"Finalize","ok":true,"outcome":"done","output":"","producer":"PR-Review","progress":{"name":"Finalize","step":12,"steps":13},"reason":"","run_id":"darn-writer-anatomy","state":"running","trace_file":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/logs/trace-20261108T144135Z-9147c44ecc9fe089-388438.log"} | {"continue":true,"envelope_path":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/shed/darn-writer-anatomy/steps/c17c7565ddde1f01.json","friction":"skipped","history_length":12,"next":"Friction-Reflect","ok":true,"outcome":"done","output":"","producer":"Finalize","progress":{"name":"Friction-Reflect","step":13,"steps":13},"reason":"","run_id":"darn-writer-anatomy","state":"running","trace_file":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/logs/trace-20261108T144145Z-c17c7565ddde1f01-413337.log"}
status error: shedverbs: step Friction-Reflect interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step darn-writer-anatomy --until-stop
```

producer: Friction-Reflect
state: failed
anomaly: interrupted-stop
history_entries: 12
trace_file: /home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/logs/trace-20261108T144222Z-4b42ba2d5e2c5b10-501308.log

history:
- Preflight / done / 2026-11-08T13:24:02Z
- Record-Write / done / 2026-11-08T13:24:03Z
- Drift-Baseline / skipped / 2026-11-08T13:24:04Z
- Drift-Await / skipped / 2026-11-08T13:24:07Z
- Drift-Expect / skipped / 2026-11-08T13:24:08Z
- Darn-Write / done / 2026-11-08T14:41:28Z
- Drift-Verify / skipped / 2026-11-08T14:41:29Z
- Code-Bouncer / done / 2026-11-08T14:41:32Z
- Drift-Recheck / skipped / 2026-11-08T14:41:38Z
- Describe / skipped / 2026-11-08T14:41:40Z
- PR-Review / done / 2026-11-08T14:41:44Z
- Finalize / done / 2026-11-08T14:42:02Z
