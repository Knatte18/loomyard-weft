# Log a running run whose waiting process is gone

Batten writes the idle notice "no agent activity readable" only when `shuttleengine.ReadAgentActivity` returns no live run for the task worktree.
On 2026-10-23 `loom-tail-review` got that notice while its Plan writer was working.
The fork-activity fix (c5d715860) closes a different gap, which yields "agents idle" instead.

`ReadAgentActivity` drops a run directory on four paths: an unreadable record, an absent record, a record that is not live, and an engine the set does not hold.
The unreadable-record and unknown-engine paths already log once per episode.
A record that reads `running` while the process recorded as waiting on it is dead was skipped silently, although that is the one skip that can hide an agent that is still working.

`ReadAgentActivity` still skips such a run.
It now logs it at Warn the first time, at Debug while the skip repeats, and at Info once the record reads live again or no longer reads `running`.
The line names the run directory, strand and pid, so the trace of Batten or the loop watchdog shows whether a later notice took this path.
The package doc's agent-activity paragraph in `internal/shuttleengine/doc.go` describes the log.

## Design decisions

- The change adds logging and does not claim to fix the incident.
  `loom-tail-review` ran on machine `osl-1046`.
  Its shuttle run directories, trace logs and that day's creel journal are not on this machine, and its tracked records in the weft repository hold no shuttle run state, so it can't be replayed or traced to one path.
- Which runs count as live is unchanged, and so are Batten's quiet judgement and the notice text.
  Counting a waiterless run as live, or widening the reader's return to count skipped runs, would change behavior or every caller's API on a guess.
- The new episodes have their own `logger.EpisodeSet` keyed by run directory.
  The unreadable-record episode for the same directory recovers on every readable read, so a shared key would end the new episode on each poll and repeat the Warn.
- A record with no pid is not logged: it predates the pid field, so a missing pid says nothing about a lost waiter.
- The behavior is a new subtest of `TestReadAgentActivity`.
  It pins one Warn and Debug lines over repeated reads, one Info recovery once the waiter is live, and no line for a pid-less or finished record.

## Check by hand

- On the next "no agent activity readable" notice, look in Batten's trace for `shuttle: skipping a run that reads running but whose waiting process is gone`.
  If the line is absent, the run was hidden another way, for example by a record already reading a terminal outcome while its agent kept working.
