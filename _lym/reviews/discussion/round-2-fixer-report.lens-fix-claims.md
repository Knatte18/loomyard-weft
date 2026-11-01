```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked every sentence the fix added or rewrote that states a bound, behaviour or fact about the code against HEAD. One claim fails and one is overstated.

**MEDIUM** — D6, added sentence: "A chair timeout while a writer has already ended the step is decided by that writer's mapping, since `EndedStep` wins", and the matching D6 acceptance clause "a chair timeout after a writer ended the step is decided by the writer's mapping".
- **Summary:** at HEAD, `EndedStep` does not win when the chair has also ended by the time `join` runs, so this mapping alone cannot deliver the claim.
- **Evidence:** in `internal/seatengine/engine.go` `join` (around :357-372), `case <-ender.signal:` first tries `case end = <-chairDone:`. It sets `stepEnded` and calls `ender.advisor.markEnded()` only in the `default` branch. Its own doc says "a chair that also ended by then decides the step as usual".
- **What breaks:** say a writer times out and the chair times out before `join` looks. The outer select may also take `chairDone` first. In both cases `EndedStep` is never set, and `mapOutcome`'s timeout branch returns the chair's error. The claim holds only if D5 or D6 also changes `join` so that a writer's end marks `EndedStep` whatever the chair did. Neither decision says that.

**LOW** — D8, added sentence: "read from the caller's strand name, resolved from the calling process's pane as creel resolves a sender", together with the Bound "a part writer … cannot reach it".
- **Summary:** creel identifies a caller by its pane only on Linux. Elsewhere it trusts the environment, so the chair-only bound is weaker than the Bound sentence says.
- **Evidence:** the `resolveSender` doc in `internal/creelengine/identity.go:44-45` reads "On Linux a peer inside a live strand's pane process tree is that strand … Elsewhere identity is the environment alone." `resolveSender` is also unexported and runs in the daemon from the connecting peer's PID. A loom verb needs a new path to reach it, and D8 does not name one.

The other claims I checked hold:
- **goto then resume is refused by the running-state check:** `Goto` sets `StatePaused` (`internal/shedengine/goto.go:124`, `:244`).
- **A halted run keeps `CurrentProducer`:** shown in `internal/shedengine/run.go`.
- **A writer timeout ends the step:** `watch` calls `ender.end` on every end of a required seat that was not accepted, a timeout included.
- **A writer that fails to start sets `EndedStep` with `StartError`:** per the `Run` doc at `engine.go:208`.
- **`Plan-Write`'s chair timeout maps to an error:** the `mapOutcome` case at `multillm.go:169`.
- **Commit on a gate-failed Stuck:** `planwrite.go` `Call`.

Tool calls:
- `sed` of `engine.go` `watch` and `join`: my context held only grep lines, not their bodies.
- `grep` of `goto.go` for state transitions: my context lacked what state `goto` leaves a run in.
