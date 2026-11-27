# Say a timed-out chair was stopped and that resume starts the row over

When a multi-seat step's chair session hits its time limit, the seat engine's `join` stops the chair before the step ends.
The Stuck reason the `MultiLLMProducer` returned still said `lym loom resume` "re-attaches the session while it is live",
so a reader expected the timed-out session's context to be recoverable;
a resume instead always finds the chair gone and starts the row in a fresh session.

The reason now reads, for example:

```
Darn-Write (seats): the chair's session timed out and was stopped; elapsed 3h0m1s, limit 3h0m0s, last session state busy (cause turn); way forward: run lym loom resume, which starts the row over in a fresh session
```

## Changes

- `internal/shedadapters/multillm.go`: the timeout reason says the session was stopped, and `timeoutWayForward` says resume starts the row over in a fresh session.
- `internal/shedadapters/multillm_test.go`: the three timeout rows of the outcome-mapping table pin the new wording.
- `internal/shedadapters/doc.go`: the package doc states that a timed-out chair is already stopped when its reason is written.

## Design decisions

- The wording follows the behaviour rather than the other way round.
  Leaving a busy timed-out chair live for a later resume would turn the time limit into a soft limit and leave an unbounded session running unwatched between steps;
  that is a change to the limit's meaning, not a wording fix.
- Whether a chair's wait on its own background work should count against its limit is left unchanged.
  The limit is the bound on a step's wall time, and exempting background waits needs a separate design of what bounds them instead.
- The writer seats' way forward is out of scope and unchanged.

## Merge danger

**Door:** two-way
**Blast radius:** shedadapters
