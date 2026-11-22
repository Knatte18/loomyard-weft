loom halted failed at Harden-Burler-Review

```
loop-exited: loop 6e08998f53a85667 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/.lym/shed/harden-fabricengine/steps/loop.log; step Harden-Burler-Review was in flight under trace f091426282814c37; loop exit: lym shed: loop 6e08998f53a85667 received signal terminated and ends without an envelope | lym shed: loop 6e08998f53a85667 process 10244 exited: signal: terminated; loop log: time=2026-11-22T11:44:23.170+01:00 level=WARN msg="shed: loop received a terminating signal and ends without an envelope" trace=0cb5bdb053691401 loop_id=6e08998f53a85667 signal=terminated; step stderr: time=2026-11-22T11:44:11.052+01:00 level=WARN msg="shuttle: could not type the segment color" trace=f091426282814c37 strandGUID=37bf58b7858eb7753a7d29d1032563e8 color=orange error="shuttle: a pause covers the session: strand \"37bf58b7858eb7753a7d29d1032563e8\": pause: a hub-wide pause has held this run since 2026-11-22T11:44:08+01:00; way forward: run \"lym tender resume\", then retry the send" | lym: held by a hub-wide pause since 2026-11-22T11:44:08+01:00; it continues after "lym tender resume"; stopped shuttle runs: ly:harden-fabricengine:harden-burler-review
status error: shedverbs: step Harden-Burler-Review interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step harden-fabricengine --until-stop
```

producer: Harden-Burler-Review
state: failed
anomaly: interrupted-stop
history_entries: 6
trace_file: /home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/.lym/logs/trace-20261122T104354Z-f091426282814c37-10260.log

history:
- Preflight / done / 2026-11-22T09:48:49Z
- Record-Write / done / 2026-11-22T09:48:55Z
- Harden-Bouncer / stuck / 2026-11-22T09:50:34Z / routed to Harden-Burler-Review
- Harden-Burler-Review / stuck / 2026-11-22T10:02:39Z / routed to Harden-Burler-Fix
- Harden-Burler-Fix / stuck / 2026-11-22T10:25:14Z / routed to Harden-Bouncer
- Harden-Bouncer / stuck / 2026-11-22T10:26:28Z / routed to Harden-Burler-Review
