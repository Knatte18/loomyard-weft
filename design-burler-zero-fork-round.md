# Burler review round hard-fails when the chair session finishes without spawning its fork reviewers

`burler-zero-fork-round` · Note · bug

Imported from [issue #756](https://github.com/Knatte18/loomyard/issues/756).

## What happened

A `lyx loom run` halted `failed` at `Plan-Burler-Review` (anomaly `producer-hard-failure`) with:

`shedadapters: Plan-Burler-Review (burler-review): round 1 attempt 1: review: burler: the cluster round did not produce exactly the requested number of fork reviewers (requested 3, spawned 0)`

The trace shows the review round starting with `clusterFan=review-plan forkCount=3` and `forkSubagents=true`.
About three and a half minutes in, the shuttle logged `session state ... state=idle-stalled cause=interrupt` followed by `shuttle: session state disagrees` (`loop=running`, `stands=loop`).
The session went `busy` again a second later and, eleven seconds after that, reached `idle-done cause=done`.
The shuttle run then finished `outcome=done` and the burler logged `review joined reviewOutcome=done` and `seatengine: step joined chairOutcome=done advisors=0`, yet no fork reviewer had been spawned.

## Why it is abnormal

The chair session reported a clean `done` while the round had spawned none of the 3 requested forks, so the run halted `failed` instead of retrying the round or reporting why the chair stopped short.
The interrupt and the state disagreement immediately before the finish suggest the chair's turn was cut short and its `done` was accepted without the fork count being checked first.
The halt message gives no indication of whether the chair never tried to fork, tried and failed, or was interrupted.

## What to do

- Check the fork count before accepting a chair `done`, and treat a chair that ended with zero forks as a stalled round that can be retried or nudged, not a hard failure.
- Record in the halt note why the chair ended (interrupt, done without forks) so the cause is distinguishable.
- Look at how an `idle-stalled cause=interrupt` followed by a quick `done` is handled in the shuttle's session-state reconciliation.

Provenance: task `ubuntu-2404-support`; notes: `loom-halt.md`.
