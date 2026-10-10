---
round: 3
focus:
  - "Cards 9, 11 and 12, the arming refusal of `lyx shed step <run-id> --until-stop`: `internal/shedcli/cli.go`'s `resolvePersistentPreRun` arms in the persistent pre-run before any verb body, so settle whether a refusal there (missing hub landing config, unreadable seed) is turned into a loop stop with a `loop` object and a `failed` status file by some planned card, and whether a planned test drives the real verb surface rather than an in-process runner."
  - "Card 12, `runUntilStop` and `waitLoopEnvelope`: `internal/lock` probes acquire the lock momentarily, so settle whether a waiter that spawned nothing and read a held lock can, on a later tick, declare a loop gone that never existed or is still arming, and whether the plan bounds that."
  - "Card 12, envelope retirement with two live waiters on one loop: settle whether the second waiter prints the same envelope or can read lock free, no envelope, no pid file and report a loop-exited stop."
  - "Card 18, `EndSession`'s order of killing the loop, awaiting the run lock and ending the reed session: settle whether a driver still alive in that window can start a new detached loop that survives the session end, against the decision that no step starts once the session-end step begins."
  - "Card 9, the `step arming refused` row: settle whether every arming error routed through the bootstrap kind, including the unknown-recipe lookup and the seed read, names a way forward in its message, as `PATTERN-refusal-way-forward` requires, and whether a planned test asserts it."
---

Focus is limited to where the loop, waiter and teardown cards interact; the remaining cards were found sound in round 2.
