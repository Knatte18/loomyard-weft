# Chair timeout stops the session, but its way forward says resume re-attaches it

`chair-timeout-way-forward` · Task · bug, shuttle, shed

Imported from [issue #281](https://github.com/Knatte18/loomyard/issues/281).

## What happened

A `Darn-Write` row halted `blocked` when its chair hit the 3h limit while it was still working:

```
Darn-Write (seats): the chair's session timed out; elapsed 3h0m1s, limit 3h0m0s, last session state busy (cause turn); way forward: run lym loom resume, which re-attaches the session while it is live and otherwise starts the row over
```

The same step had already stopped that session before it printed the way forward.
From the run's trace, in order, with paths and ids left out:

```
15:50:09.560 shuttle: run ended with background shells outstanding; the strand is left to its caller
15:50:09.561 shuttle: run finished outcome=timeout cleanedUp=false
15:50:09.578 shuttle: stopping a strand live=true
15:50:10.381 seatengine: step joined chairOutcome=timeout
15:50:10.411 shed: step done producer=Darn-Write outcome=stuck state=blocked
```

The next step, one second later, started a new `darn` session with a new session id.

## Why it is abnormal

`timeoutWayForward` in `internal/shedadapters/multillm.go` says `lym loom resume` re-attaches the session while it is live.
`Engine.join` in `internal/seatengine/engine.go` stops a chair that timed out, so after this halt the session is never live unless the stop itself failed, which returns an error instead of this reason.
A reader of the halt expects the three hours of session context to be recoverable, and it is not: the row starts over.

The stopped session was busy and had polled background work for much of its run, so the limit ended a working session rather than a hung one.

## What to do

- Make the timeout reason say that the session was stopped and that `lym loom resume` starts the row over.
- Alternatively, if re-attaching is the intended behaviour, leave a timed-out chair that is still busy live for resume to pick up.
- Check the tests in `internal/shedadapters/multillm_test.go` that pin the current wording.
- Consider whether a row waiting on its own long background work should count that wait against the chair's limit.

---

Provenance: task `weaver-bench`; notes: `loom-halt.md`.
