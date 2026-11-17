loom halted failed at Darn-Write

```
shedadapters: Darn-Write (seats): seat run: seatengine: chair wait: shuttle: reed did not track strand "a47528574954381c6e859804677b1411" on 2 consecutive liveness checks: reed's strand table no longer holds this run's strand — reed's bookkeeping was reset under the run (a reed remove/down, a lost or rebuilt reed.json, or a worktree renamed while the run was in flight), which says nothing about the agent: its process may still be working in its pane. Check "lym reed status"
```

producer: Darn-Write
state: failed
anomaly: producer-hard-failure
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/cwb-codex/.lym/logs/trace-20261117T173158Z-1f9751fd77af3733-3323997.log

history:
- Preflight / done / 2026-11-17T17:31:48Z
- Record-Write / done / 2026-11-17T17:31:50Z
- Drift-Baseline / skipped / 2026-11-17T17:31:58Z
- Drift-Await / skipped / 2026-11-17T17:32:01Z
- Drift-Expect / skipped / 2026-11-17T17:32:03Z
