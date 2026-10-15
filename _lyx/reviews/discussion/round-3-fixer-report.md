# Fixer report — discussion, round 3

## Changes

- F1 (LOW, decision): fixed as proposed.
  The premise holds: `Run` in `internal/websterengine/runlevel.go` creates the scratch dir and then tries `run.lock`, returning `ErrRunBusy` when it is held, so a removal placed literally first would run in a refused `Run`.
  - D2 now says `websterengine.Run` removes the record once it holds run.lock and before the auto-rebaseline, so a Run refused with `ErrRunBusy` never deletes the live Run's record; the Rationale's "before anything else at Run entry" now reads the same way.
  - D2 gains one sentence: a removal failure, at Run entry or at begin-batch, is logged at Warn and changes neither the refusal nor the Run outcome.
  - Open risks gains the bullet that a record surviving a failed removal can route a later stuck outcome of the same Webster call, ending at the next successful removal at Run entry or begin-batch.
  - The acceptance criterion on Run's removal now places it after run.lock and pins that a Run refused with `ErrRunBusy` leaves the record in place.
  - The support log's Q2 ledger entry names the run.lock placement.

## Disputed

None.

## Deferred

None.
