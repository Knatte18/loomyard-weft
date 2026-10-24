# Log a running run whose waiting process is gone

Batten's idle notice "no agent activity readable" appears only when `shuttleengine.ReadAgentActivity` finds no live run.
A writer in `loom-tail-review` was working when the notice was quoted, so a live run was hidden from the reader.
The incident's run directories and traces are not available, so the silent path it took cannot be shown.
This change makes the most likely one visible instead of claiming a fix.

## What changes

`ReadAgentActivity` still skips a run whose record reads `running` while the pid of the process recorded as waiting on it is dead.
It now logs the skip, where it logged nothing before:

- one Warn at the first skip, naming the run directory, strand and pid;
- Debug lines while the skip repeats;
- one Info when the record reads live again, reads anything other than `running`, or is gone.

Which runs count as live, the notice text and Batten's quiet judgement are unchanged.
A record with no pid predates the field and says nothing about a lost waiter, so it logs nothing.
A finished record logs nothing either.
`doc.go` of `shuttleengine` records the new behavior.

## Design decisions

- The episode set is its own, `waiterlessRuns`, keyed by run directory.
  The unreadable-record episode of the same directory recovers on every readable read, so sharing its key would end the new episode on each poll and repeat the Warn.
- The read runs in Batten's and the loop watchdog's polls, so the line follows `PATTERN-log-levels`: Warn once, then Debug.
- The run record's shape is untouched, and the change stays in the provider-neutral reader.

## To check

The next occurrence of the notice tells which path it was:
a Warn from this change means a waiter died while its agent kept working;
no such Warn points to another silent path, such as a record already reading a terminal outcome while its agent worked.
