# shuttleengine: log a running run whose waiter process is gone

Batten's idle notice "no agent activity readable" appears only when `ReadAgentActivity` finds no live run at all.
A run is live when its record reads `running` and the pid of its recorded waiter is alive.
A run whose waiter died while its agent kept working is skipped without a trace, so the activity reader sees nothing and Batten reports the notice for a working agent.
The `loom-tail-review` incident that prompted this cannot be replayed from the records on hand, so this change makes the next occurrence show its cause instead of claiming a fix.

## What changes

`ReadAgentActivity` still skips such a run, and now logs it once per episode, keyed by run directory:

- Warn at the first skip, naming the run directory, strand and pid.
- Debug on each repeated skip.
- Info when the record reads live again, no longer reads `running`, or has no record.

Which runs count as live, the notice text and Batten's quiet judgement are unchanged.
`doc.go` for `shuttleengine` documents the new logging.

## Design decisions

- The episode set `waiterlessRuns` is separate from `unreadableFiles`, because the unreadable-record episode of a run directory recovers on every readable read and would end the new episode on each poll, repeating the Warn.
- A record with no pid is not logged: it predates the pid field, so a missing pid says nothing about a lost waiter.
- The run record's shape is unchanged, and the change stays in the provider-neutral reader.

## To check

- If the incident took another silent path, such as a record already reading a terminal outcome while its agent kept working, this change does not show it.
  Absence of the new Warn in the next occurrence's trace points there.
- Nothing was run against a live run with a killed waiter; the new tests in `activity_test.go` cover the episode's Warn, Debug and recovery lines.
