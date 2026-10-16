# Round 2 fixer-report — discussion

## Changes

- **F1** (fixed as proposed): D5's residual-window paragraph now says `revive` reads a session's coverage and sends its line while holding the pause record's write lock, the lock `pause` takes to add its entry.
  A pause entry is either live at the read, so the session gets no line, or added after the line lands, when the grace poll reads the session busy and the interrupt pass records it.
  The Bound states the cost: one conditioned send at a time, so a concurrent `pause` waits at most one send.
  Coverage reads (`pauserecord.Read`) take no lock, so the send's own coverage reads never wait on it; I checked this so the lock can't deadlock.
  New Acceptance criterion: a pause added while a `revive` line is in flight interrupts and records that session.
- **F2** (fixed as proposed): D5's bullet now says `shuttleengine` gains a conditioned send, one whose ready reading must show a named state and cause, which refuses at once with no idle wait otherwise.
  Its Bound says it only adds a refusal to today's send.
  D5's overall Bound names the conditioned send.
  Scope In lists the conditioned send, and the Docs constraint names `shuttleengine`'s `doc.go`.
- **F3** (fixed as proposed): D6 now says a cancellation that arrives before the caller's own deadline passes still ends the call.
  Once that deadline has passed, only the moved deadline ends it, so the extension is the longest a late cancellation can be ignored.
  The cancellation Acceptance criterion now says "after release, before its original deadline passes".
- **F4** (fixed as proposed): D8's Bound now says the new image holds the cycle machine under a live hub entry from its first tick, and interrupts a busy orch only for an entry whose grace end is still ahead.
  An entry whose grace ended in the gap leaves the orch uninterrupted, which the no-live-watcher report already says.

The support log needed no change.

## Disputed

None.

## Deferred

None.
