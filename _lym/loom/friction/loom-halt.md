loom halted failed at Harden-Burler-Review

```
loop-exited: loop 31bbf1ebccfde952 ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/.lym/shed/harden-fabricengine/steps/loop.log; step Harden-Burler-Review was in flight under trace 88b20981282d7b3c; loop exit: not recorded, since no process marked the loop's exit, and the machine has booted since the loop started, so a stop of the machine likely ended it; stopped shuttle runs: ly:harden-fabricengine:harden-burler-review
status error: shedverbs: step Harden-Burler-Review interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step harden-fabricengine --until-stop
```

producer: Harden-Burler-Review
state: failed
anomaly: interrupted-stop
history_entries: 6
trace_file: /home/knatte/Code/loomyard-LYMHUB/harden-fabricengine/.lym/logs/trace-20261122T102634Z-88b20981282d7b3c-3276796.log

history:
- Preflight / done / 2026-11-22T09:48:49Z
- Record-Write / done / 2026-11-22T09:48:55Z
- Harden-Bouncer / stuck / 2026-11-22T09:50:34Z / routed to Harden-Burler-Review
- Harden-Burler-Review / stuck / 2026-11-22T10:02:39Z / routed to Harden-Burler-Fix
- Harden-Burler-Fix / stuck / 2026-11-22T10:25:14Z / routed to Harden-Bouncer
- Harden-Bouncer / stuck / 2026-11-22T10:26:28Z / routed to Harden-Burler-Review
