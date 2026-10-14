# Round 3 fixer-report — discussion

## Changes

- F1 MEDIUM, fixed as proposed.
  Premise confirmed: `handOver` in `internal/loomcli/drive.go` appends the `hand-over` settle record and then calls `handBack`, which writes no settle record, while `handBackAs` appends a `hand-back` record before its park marker.
  D9's exemption now reads that a park marker does not end the hold while the newest settle record at this stop key later than the stop is the `hand-over`, and a successor's `hand-back` record lets its marker end it.
  Added the acceptance criterion for a successor tackler's hand-back with no arbiter attempt.
- F2 LOW, fixed as proposed.
  Premise confirmed: `readStopReport` in `internal/battencli/wire.go` reads only the park marker, and `startReport` writes report files before the tackler and the arbiter run.
  D9's end condition names the driver's park marker only and says the `startReport` files do not end the hold;
  the `stateChangeReady` bullet's reason is now that the report wait must start no earlier than the hold's end, and its follow-on sentence names the park marker.
  The hold-ending criterion and the hand-on criterion name the park marker instead of a stop report.
- F3 LOW, fixed as proposed.
  Premise confirmed: `loomengine.ArbiterAttempt` carries `SpawnedAt` and `Deadline` but no hand-on time.
  D9's `stateChangeReady` bullet names the end's source per condition: the park marker's time, the hand-on verdict file's time, the backstop's computed time, and batten's first observation for an applied resume or the arbiter switched off, with a restart taking its own start time.
  The Bound's restart sentence covers the end, and the restart case is now its own criterion covering start and end.
- Support log: a round 3 entry and Q14 for the end-time source.

## Disputed

None.

## Deferred

None.
