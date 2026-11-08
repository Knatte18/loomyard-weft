loom halted failed at Darn-Write

```
loop-exited: loop afd778dbf6d7a705 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/shed/darn-writer-anatomy/steps/loop.log; loop exit: not recorded; loop log: {"continue":true,"envelope_path":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/shed/darn-writer-anatomy/steps/0c66aa9c171efa72.json","friction":"skipped","history_length":1,"next":"Record-Write","ok":true,"outcome":"done","output":"","producer":"Preflight","progress":{"name":"Record-Write","step":2,"steps":13},"reason":"","run_id":"darn-writer-anatomy","state":"running","trace_file":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/logs/trace-20261108T132400Z-0c66aa9c171efa72-4192088.log"} | {"continue":true,"envelope_path":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/shed/darn-writer-anatomy/steps/5bd21b9b7404a38b.json","friction":"skipped","history_length":2,"next":"Drift-Baseline","ok":true,"outcome":"done","output":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/_lym/discussion/decision-record.md","producer":"Record-Write","progress":{"name":"Drift-Baseline","step":3,"steps":13},"reason":"","run_id":"darn-writer-anatomy","state":"running","trace_file":"/home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/logs/trace-20261108T132403Z-5bd21b9b7404a38b-4192415.log"}
status error: shedverbs: step Darn-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step darn-writer-anatomy --until-stop
```

producer: Darn-Write
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/darn-writer-anatomy/.lym/shed/darn-writer-anatomy/steps/loop.log

history:
- Preflight / done / 2026-11-08T13:24:02Z
- Record-Write / done / 2026-11-08T13:24:03Z
- Drift-Baseline / skipped / 2026-11-08T13:24:04Z
- Drift-Await / skipped / 2026-11-08T13:24:07Z
- Drift-Expect / skipped / 2026-11-08T13:24:08Z
