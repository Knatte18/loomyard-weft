# Dobby counts a just-ended agent run in the no-run idle clock

Dobby's quiet notice could report a healthy run as idle for its whole length.
When dobby reads no running shuttle run for a child, the idle clock falls back to the later of the child's newest history entry and its state episode's since.
In the seconds between a session ending (shuttle sets `run.json`'s outcome to `done`) and the row writing its history entry, both of those still read the row's start.
So a probe in that gap read a row older than the quiet window as idle since the row began, and sent `no agent activity readable for 1h14m30s` for a run that had just finished normally.

## What changes

- `shuttleengine.ReadNewestRunEnd` (`internal/shuttleengine/activity.go`) returns the modification time of the newest run record under the run-directory root whose outcome reads anything but running.
  A run's finalize is its record's last write, so that time is when the session ended.
  It reads files only, skips a directory with no record, logs an undecodable record once per episode as `ReadAgentActivity` does, and returns the zero time for an absent root or no ended run.
- `dobbyengine.RunNoticeDeps` gains a `NewestAgentEnd` seam.
  It is read only when `Agents` reads no running run; a nil seam reads none, and a read error is logged once per episode and sends no activity notice, as an `Agents` error does.
- `judgeAgents`'s no-run fallback now counts the latest of the newest history entry, the newest ended run's end and the state's since.
  The quiet episode stamp moves with it, so a session ending opens a new quiet episode.
- `battencli` wires the seam through `readNewestAgentEnd`, which loads the task worktree's shuttle config and calls `ReadNewestRunEnd`.

## Design decisions

- A separate function and seam rather than having `ReadAgentActivity` return ended runs too.
  `ReadAgentActivity`'s other callers in `loomcli` treat every reading as a running run, and a run-notices reading of an ended run as a run would turn the fallback's `no agent activity readable` into the `agents idle` path.
  The seam is called only in the no-run case, so a normal probe pays nothing extra.
- The record's modification time stands for the end time instead of re-reading each ended run's events and transcript: it is one stat per run directory, and finalize writes the record after the session's last activity.
- The notice text is unchanged.

## Tests

- `TestReadNewestRunEnd` covers no runs, a running run only, the newest of several ended outcomes winning, a newer running record ignored, a directory with no record and an undecodable record skipped, and an absent root.
- `TestRunNotice_Conditions` gains rows for a just-ended run holding the quiet notice, the fallback measuring from the ended run when it is newer than the history entry, and an unreadable ended-run reading sending nothing.
- `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` asserts the new seam refuses an absent task worktree naming the slug.
