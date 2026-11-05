# Count gate fixers, gate runs and run hand-over in quiet notice

Dobby sent "no agent activity readable" notices for darn runs that were working.
The quiet check read only the writer's strand, so it missed two kinds of activity:

- A gate-fixer ran in its own session while the writer's strand sat idle.
- A `lym gate test` ran, or a writer's shuttle had just ended and the next row had not started.
  A done run's cleanup removes its run directory, so no record of the end was left to read.

Each false notice cost the orch a stop investigation that found nothing to do.
The quiet check now counts these as activity, and a notice fires only when none of them shows activity for the threshold.

## What changes

- `battencli`: `readAgentActivity` reports a live wait when `gateProcessLive` finds a registered process of the pair's slug.
  A `lym gate` runner counts at any age.
  A fixer's pane process counts only while it is younger than the fixer's timeout:
  `websterengine`'s recovery timeout for a verify fixer, the gate config's fixer timeout for any other.
  A process of another slug or another role does not count.
- `shuttleengine`: every `finalize` stamps a `run-ended` file at the run-directory root, before a done run's cleanup removes the run directory.
  `ReadNewestRunEnd` returns the later of that stamp and the newest ended run record, so the end of a removed run stays readable and covers the hand-over gap.
- `dobbyengine`: the `Agents` doc comment names the new waits; no behavior change.

## Design decisions

- The age bound exists because a fixer that ended without its strand being removed leaves a pane process that outlives its work.
  Without the bound, such a leftover would silence the notice for good.
- A fixer with no known bound counts as live.
  That happens only when the gate config is absent, which a gate fixer cannot be running without.
- The stamp is a separate file rather than a keep-alive of the run directory, so cleanup of a done run is unchanged.
  A stamp failure is logged and non-fatal.

## To check by hand

- A fixer pane process that ends without its strand removed no longer holds the quiet notice past the fixer's timeout.
  The fixer's record age is its registration time, so check that a fixer which legitimately runs longer than its configured timeout is still caught by the notice, as intended, rather than silenced.
- The test for another slug's gate process pins that it does not count.
  Check that the row covers the pair's slug as `readAgentActivity` derives it from `taskLocation.WorktreeName`.
- Not run: a live darn run through a real gate-fixer window and a `lym gate test`, to see no quiet notice fire.
