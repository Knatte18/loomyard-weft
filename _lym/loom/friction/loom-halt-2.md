loom halted failed at Discussion-Burler-Review

```
loop-exited: loop f09ec48507d22feb ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/orch-full-autonomy-mode/.lym/shed/orch-full-autonomy-mode/steps/loop.log; step Discussion-Burler-Review was in flight under trace 688f2eaf3be32b72; loop exit: lym shed: loop f09ec48507d22feb received signal terminated and ends without an envelope | lym shed: loop f09ec48507d22feb process 12063 exited: signal: terminated; loop log: time=2026-11-22T11:44:23.171+01:00 level=WARN msg="shed: loop received a terminating signal and ends without an envelope" trace=13d8e84501935476 loop_id=f09ec48507d22feb signal=terminated; step stderr: time=2026-11-22T11:44:17.417+01:00 level=WARN msg="shuttle: could not type the segment color" trace=688f2eaf3be32b72 strandGUID=465708b08c55a1ead9d8d672369feef5 color=orange error="shuttle: a pause covers the session: strand \"465708b08c55a1ead9d8d672369feef5\": pause: a hub-wide pause has held this run since 2026-11-22T11:44:08+01:00; way forward: run \"lym tender resume\", then retry the send" | lym: held by a hub-wide pause since 2026-11-22T11:44:08+01:00; it continues after "lym tender resume"; stopped shuttle runs: ly:orch-full-autonomy-mode:discussion-burler-review
status error: shedverbs: step Discussion-Burler-Review interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step orch-full-autonomy-mode --until-stop
```

producer: Discussion-Burler-Review
state: failed
anomaly: interrupted-stop
history_entries: 5
trace_file: /home/knatte/Code/loomyard-LYMHUB/orch-full-autonomy-mode/.lym/logs/trace-20261122T104400Z-688f2eaf3be32b72-12077.log

history:
- Discussion-Write / done / 2026-11-22T10:04:12Z
- Discussion-Bouncer / stuck / 2026-11-22T10:04:40Z / routed to Discussion-Burler-Review
- Discussion-Burler-Review / stuck / 2026-11-22T10:13:04Z / routed to Discussion-Burler-Fix
- Discussion-Burler-Fix / stuck / 2026-11-22T10:20:46Z / routed to Discussion-Bouncer
- Discussion-Bouncer / stuck / 2026-11-22T10:21:19Z / routed to Discussion-Burler-Review
