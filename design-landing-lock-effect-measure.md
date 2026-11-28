# Measure how much landing-lock-only-for-land cut the wait for the parent lock

`landing-lock-effect-measure` · Note · research, landing, batten

Depends on: [`finalize-owed-tips-gate`](design-finalize-owed-tips-gate.md)

## Why

`landing-lock-only-for-land` (`d87a9ce24`) made Finalize merge and gate with no parent lock held, and take the lock only to land.
Whether ready runs now land without queuing is unmeasured.
On 2026-11-28 `reset-partial-commit-push` still showed 38 minutes at Finalize with no agent activity before it took the lock on `main`.
The answer decides whether the landing queue note `landing-queue` is still worth building.

## What to measure

From the drive reports and gate run logs of landings already made, no new runs:

- For each landing, the time from Finalize's start to taking the parent lock, the time holding it, and the lost races (`landing.finalize_lost_races`).
- How often a run fell back to merging and gating under the lock.
- The same figures for the landings before `d87a9ce24`, for comparison.

Cap: the last 20 landings before `d87a9ce24` and the first 20 after `finalize-owed-tips-gate` lands, on the prime of this hub.
The report lands as a commit under `docs/research/`.
