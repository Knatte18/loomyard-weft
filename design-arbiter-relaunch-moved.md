# Arbiter relaunch refused when the run already moved to the verdict row

`arbiter-relaunch-moved` · Note · bug, loom, shed

`operator-followups`, 2026-10-12: Merge-In verify failed after merging the parent, and the driver ran the arbiter (attempt 2).
17 seconds after the arbiter spawned, the run moved from Merge-In's `stuck` row to Webster-Burler-Review, the same row the arbiter's verdict later named as its goto.
The relaunch was then refused with bootstrap "the run moved since the arbiter spawned", and the run parked for the parent although the verdict was already in effect.

## What to fix

- A relaunch whose verdict goto names the row the run already stands on should be admitted, not refused on the stop-key mismatch.
- Find what moved the run during the arbiter's spawn: a second actor moving a run while its arbiter is live is a race of its own.

## Also seen

The same arbiter got an invalid-JSON error from `lyx board get operator-followups`: it likely passed a bare slug, not a JSON payload; check the arbiter stencil's wording.
