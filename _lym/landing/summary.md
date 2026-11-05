# Quiet notice counts a removed run's end, gate runs and gate fixers

Dobby sent quiet notices ("no agent activity readable") for darn runs that were working.
In each case the notice came seconds after a shuttle run finished.
A done run's cleanup removes its run directory, which held the only record of when the run ended.
`ReadNewestRunEnd` then found no ended run, so the quiet check measured idle time from the child's last history entry, often more than the quiet window old.

## What changes

- **shuttleengine**: every finalize sets a run-end stamp, the `run-ended` file at the run-directory root, before the cleanup removes the run directory.
  `ReadNewestRunEnd` returns the later of the stamp and the newest ended run record.
  A writer or fixer that has just finished now counts as recent activity while the next row starts.
- **battencli**: `readAgentActivity`, behind dobby's `Agents` seam, reports a live wait when a process registered under the pair's slug is a detached `lym gate` runner (`runproc.RoleGate`) or a cold fixer's pane process (any `gatefixer` fixer role).
  So a fixer that works while the writer's strand sits idle, or a gate run with no agent session at all, holds the quiet notice.
  A gate entry run inside a writer's shuttle gate was already a live wait through its `gate <entry>` wait marker.

The quiet notice still fires when none of these shows activity for the window.

## Design decisions

- The end stamp is one file at the run-directory root rather than a record kept per run.
  Only the newest end matters to the reader, and the root already holds `engines-run.json`, which every run-directory scan skips as a non-directory.
- A live fixer or gate process counts as a live wait, not as activity with a time.
  Both are bounded by their own timeouts: the fixer timeout and the gate's CPU watchdog.
  So a hung one delays the notice by that bound at most.

## Tests

- `TestReadNewestRunEnd` gains rows for a stamp with no record left, a stamp newer than every record, and a record newer than the stamp.
- The `Wait` cleanup table asserts that a cleaned-up done run still leaves its end readable through `ReadNewestRunEnd`.
- `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` registers this process under the pair's slug as a loop (no wait), a gate runner, a gate fixer and a numbered verify fixer (each a wait).
