# Repair record: interrupted Webster-Burler step

```yaml
failure: `lyx shed step test-tiers` ended on SIGQUIT with a goroutine dump and no envelope (trace 3a0fb77eaeb44afd), during Webster-Burler round 2; signal sender unknown
status_read: current_producer Webster-Burler, history_length 28, state running, interrupt_policy reinvoke, last_step finished false
action: re-invoked the step once (repair 1 of 2 for this row)
outcome: trace 0b55c94e99acc182 advanced the run to history_length 29, next Webster-Bouncer
```

Trace lines acted on:

- `burler: round starting ... round=2 ... reviewPath=.../_lyx/reviews/webster/round-2-review.md` (interrupted trace, no mutation record, no `shed: step done`)
- `shedadapters: burler round producer reached with the highest complete round unjudged; handing back for judgment instead of running a fresh round ... round=2` (re-invoke trace)
