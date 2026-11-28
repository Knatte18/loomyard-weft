# Webster run-exit audit flags Master friction note as a parent write

`webster-audit-master-friction` · Note · bug, webster

## Seen (2026-11-28, `kde-window-placement-build`)

At a Webster run exit, the audit raised `parent-write` against `_lym/loom/friction/webster-master.md`, changed since Master spawned.
That file is Master's own friction note: `_lym/webster/state.json` lists it under `masterNotePaths`, and the finding's own text says Master may write its friction note.
The way forward it named, restoring the file to the records commit, deletes the note, since that commit does not hold it; `lym webster accept-audit` refuses while the file differs.
So the run lost its friction note to clear a false finding.

## Idea

The parent-write audit admits every path in `masterNotePaths`, so a Master friction note never raises a finding.
