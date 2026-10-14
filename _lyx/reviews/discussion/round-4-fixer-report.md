# Round 4 fixer report — discussion

## Changes

- F1 (fixed as proposed): D9's end-condition bullet now says an attempt hands on when its saved `envelope.json`, written by the verb's `finish` for every outcome, carries verdict `hand-on`, and names why `verdict.yaml` is not the signal.
  The end time is the write time of that attempt's `envelope.json`, in D9, the Bound, the restart acceptance criterion and Q14.
  Premise checked: `handOn` in `internal/loomcli/arbiter.go` routes through `finish`, which writes `envelope.json` for every outcome.
- F2 (fixed as proposed): the hand-on end condition is limited to an attempt whose envelope was written in this episode, and D9 and Q14 state that the hold's end time is never earlier than the episode's since.
- F3 (fixed as proposed): the report wait's reason now names the cause, a hold outlasting `noticeReportWait` finding the wait already spent at its end.
- F4 (fixed, first option): an applied resume is dated by its attempt's `applied` mark write time (`MarkArbiterApplied` in `internal/loomengine/arbiter.go` writes that file).
  It is removed from the batten-first-observed case in D9, the restart criterion and Q14; only the arbiter switched off keeps that case.
  The Bound's list of end records names the `applied` mark too.

## Disputed

None.

## Deferred

None.
