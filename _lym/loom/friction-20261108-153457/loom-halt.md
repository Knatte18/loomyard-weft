loom halted failed at Darn-Write

```
loop-exited: loop 9e007459d172da8b ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/shed/slack-presence-auto/steps/loop.log; loop exit: not recorded; loop log: {"continue":true,"envelope_path":"/home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/shed/slack-presence-auto/steps/9a3407a81e7b07f7.json","friction":"skipped","history_length":1,"next":"Record-Write","ok":true,"outcome":"done","output":"","producer":"Preflight","progress":{"name":"Record-Write","step":2,"steps":13},"reason":"","run_id":"slack-presence-auto","state":"running","trace_file":"/home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/logs/trace-20261108T130707Z-9a3407a81e7b07f7-4125740.log"} | {"continue":true,"envelope_path":"/home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/shed/slack-presence-auto/steps/8dc8f1265986076f.json","friction":"skipped","history_length":2,"next":"Drift-Baseline","ok":true,"outcome":"done","output":"/home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/_lym/discussion/decision-record.md","producer":"Record-Write","progress":{"name":"Drift-Baseline","step":3,"steps":13},"reason":"","run_id":"slack-presence-auto","state":"running","trace_file":"/home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/logs/trace-20261108T130711Z-8dc8f1265986076f-4126407.log"}
status error: shedverbs: step Darn-Write interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step slack-presence-auto --until-stop
```

producer: Darn-Write
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/slack-presence-auto/.lym/shed/slack-presence-auto/steps/loop.log

history:
- Preflight / done / 2026-11-08T13:07:09Z
- Record-Write / done / 2026-11-08T13:07:11Z
- Drift-Baseline / skipped / 2026-11-08T13:07:14Z
- Drift-Await / skipped / 2026-11-08T13:07:18Z
- Drift-Expect / skipped / 2026-11-08T13:07:19Z
