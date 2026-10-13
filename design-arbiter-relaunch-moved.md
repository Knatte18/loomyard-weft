# Arbiter relaunch refused when the run already moved to the verdict row

`arbiter-relaunch-moved` · Note · bug, loom, shed

`operator-followups`, 2026-10-12: Merge-In verify failed after merging the parent, and the driver ran the arbiter (attempt 2).
17 seconds after the arbiter spawned, the run moved from Merge-In's `stuck` row to Webster-Burler-Review, the same row the arbiter's verdict later named as its goto.
The relaunch was then refused with bootstrap "the run moved since the arbiter spawned", and the run parked for the parent although the verdict was already in effect.

## What to fix

- A relaunch whose verdict goto names the row the run already stands on should be admitted, not refused on the stop-key mismatch.
- Find what moved the run during the arbiter's spawn: a second actor moving a run while its arbiter is live is a race of its own.

## Seen again (2026-10-13, `reed-multi-window`)

The run stopped at Merge-In on a verify failure (a test-budget row), the status file recorded a goto to Webster-Burler-Review at 19:04:25Z, and arbiter attempt 2 then answered `resume` with that same row as its goto.
The driver's re-launch was refused with kind `bootstrap`, "the run moved since the arbiter spawned", although the verdict's goto target was the row the run already sat on.
A `lyx pause` (19:04:42Z) and an overnight host sleep fell inside the arbiter's spawn, so the pause or the goto ahead of the spawn may be what counts as the move; the orch re-launched by hand.

## Also seen

The same arbiter got an invalid-JSON error from `lyx board get operator-followups`: it likely passed a bare slug, not a JSON payload; check the arbiter stencil's wording.
