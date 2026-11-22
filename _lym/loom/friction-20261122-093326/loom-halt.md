loom halted failed at Darn-Write

```
shedadapters: Darn-Write (seats): seat run: seatengine: seat "darn": parentdirective: stencilstore: stencil "parent-directive-parent" was written by lym build 43fbea87e2d5c57e5b9fee1fd97de36fceddf54b, which is newer than the running build 3a9f8d8419fbcd415992ca15a8c6e9d25822015e, and this process may not render it; way forward: restart the run's driver on the current lym binary, and update that binary first if it is older than the board
```

producer: Darn-Write
state: failed
anomaly: producer-hard-failure
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/hardener-self-charter/.lym/logs/trace-20261121T192401Z-c33a162873ce429c-2362276.log

history:
- Preflight / done / 2026-11-21T19:23:31Z
- Record-Write / done / 2026-11-21T19:23:43Z
- Drift-Baseline / skipped / 2026-11-21T19:24:01Z
- Drift-Await / skipped / 2026-11-21T19:24:17Z
- Drift-Expect / skipped / 2026-11-21T19:24:28Z
