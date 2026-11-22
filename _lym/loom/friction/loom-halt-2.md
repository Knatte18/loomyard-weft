loom halted failed at Sampler-Session

```
loop-exited: loop 0e1ca1f8ab8f3cc3 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/readme-declutter/.lym/shed/readme-declutter/steps/loop.log; step Sampler-Session was in flight under trace 2ea9630adb703268; loop exit: lym shed: loop 0e1ca1f8ab8f3cc3 received signal terminated and ends without an envelope | lym shed: loop 0e1ca1f8ab8f3cc3 process 11717 exited: signal: terminated; loop log: time=2026-11-22T11:44:23.171+01:00 level=WARN msg="shed: loop received a terminating signal and ends without an envelope" trace=b1675ff775f8d50e loop_id=0e1ca1f8ab8f3cc3 signal=terminated; step stderr: time=2026-11-22T11:44:17.745+01:00 level=WARN msg="shuttle: could not type the segment color" trace=2ea9630adb703268 strandGUID=095d8c3da2b701b5c31fda8cd37a980d color=red error="shuttle: a pause covers the session: strand \"095d8c3da2b701b5c31fda8cd37a980d\": pause: a hub-wide pause has held this run since 2026-11-22T11:44:08+01:00; way forward: run \"lym tender resume\", then retry the send" | lym: held by a hub-wide pause since 2026-11-22T11:44:08+01:00; it continues after "lym tender resume"; stopped shuttle runs: ly:readme-declutter:sampler
status error: shedverbs: step Sampler-Session interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step readme-declutter --until-stop
```

producer: Sampler-Session
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/readme-declutter/.lym/logs/trace-20261122T104358Z-2ea9630adb703268-11734.log

history:
- Preflight / done / 2026-11-22T09:57:51Z
- Record-Write / done / 2026-11-22T09:57:56Z
- Drift-Baseline / skipped / 2026-11-22T09:58:01Z
- Drift-Await / skipped / 2026-11-22T09:58:09Z
- Drift-Expect / skipped / 2026-11-22T09:58:14Z
