# Dobby counts an ended agent session as activity in the no-run idle clock

Dobby sent a false `no agent activity readable` notice for a healthy run.
Between a shuttle session ending and its row writing the history entry, `shuttleengine.ReadAgentActivity` reads no running run.
`judgeAgents` then measured idle time from the row's start, so any row older than the quiet window read as idle for its whole length.
The same gap opens between session respawns inside one row.

With no running run read, the idle clock now starts from the latest of the child's newest history entry, the end of its newest ended agent run, and the state's `since`.
A session that has just ended counts as fresh activity.

## What changes

- `shuttleengine.ReadNewestRunEnd` returns the modification time of the newest run record whose outcome is anything but `running`, or the zero time when none exists or the run root is absent.
  A run's finalize is its record's last write, so that time is the run's end.
  A directory with no record is skipped, and an undecodable record is skipped and logged once per episode, as in `ReadAgentActivity`.
- `dobbyengine.RunNoticeDeps` gains a `NewestAgentEnd` seam, wired in `battencli` over the child's task worktree.
  A nil seam reads no end time.
- `judgeAgents` takes the end time as an extra clock input.
  The seam is read only when `Agents` reads no run.
- An unreadable end time sends no notice and logs through the same failure tracker the agents seam uses, so a transient read error cannot raise a false notice.

## Design decisions

- A separate function, not a change to `ReadAgentActivity`: that reading stays limited to running runs, which `judgeAgents` also uses to tell usage-limit and API-error stalls, so ended runs stay out of those judgments.
- The end time is the record's file modification time rather than a field in `run.json`, so no record format changes.

## Check by hand

- The mod-time reading was only exercised by tests that set the time with `Chtimes`.
  Nothing ran against a live shuttle session, so watch one run's session end and respawn to confirm no `no agent activity readable` notice arrives in the gap.
