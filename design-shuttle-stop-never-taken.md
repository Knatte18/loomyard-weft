# shuttle: Wait never takes a valid Stop; a burler round's shed step stalls

`shuttle-stop-never-taken` · Note · bug

Imported from [issue #411](https://github.com/Knatte18/loomyard/issues/411).

Seen 2026-10-06 in `test-suite-prune-rest`, Webster-Burler round 2.

- `lyx shed step` (trace a1d89adb0a964012) started the burler shuttle run at 19:29:12 (run dir `.lyx/shuttle/976daeeb219e18f87d1a6954c562b64e`, session 97d818aa-8fb0-4103-9b56-0ae4da2a75b7).
- The burler ended its turn at 19:36:12: `events.jsonl` holds a valid Stop past `promptOffset`, with `background_tasks: []`, and both output files (round-2 review and fixer report) exist.
- The step never returned. 15 minutes later it had no child processes, the verify gate had not run (`.lyx/verify/` unchanged since 19:28), and the binary already included c4d924316.
- The burler's transcript ends at 19:41:38 with a `continued-in` entry pointing at session 8e5a1d83, cause unknown.

A SIGQUIT goroutine dump shows the step parked in the poll loop:

```
shuttleengine.(*Run).Wait            internal/shuttleengine/wait.go:305 (realClock.Sleep)
shuttleengine.(*Runner).RunGated     internal/shuttleengine/run.go:643
burlerengine.(*Engine).Run           internal/burlerengine/engine.go:197
shedadapters.(*BurlerProducer).Call  internal/shedadapters/burler.go:419
shedengine.(*Shed).stepLocked        internal/shedengine/run.go:232
```

So `pollEventsTick` (`wait.go:613`) kept not returning Done for a Stop that looks complete.
Worth checking whether the `continued-in` session switch, or an output-file or offset check, keeps the Stop from being taken.
The `shuttle-asking-holds` note and #407 cover a turn end read too early; this is a turn end never read.
