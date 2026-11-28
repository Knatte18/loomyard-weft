# Measure how much landing-lock-only-for-land cut the wait for the parent lock

`landing-lock-effect-measure` · Note · research, landing, batten

## Why

`landing-lock-only-for-land` (`d87a9ce24`) made Finalize merge and gate with no parent lock held, and take the lock only to land.
Whether ready runs now land without queuing is unmeasured.
On 2026-11-28 `reset-partial-commit-push` still showed 38 minutes at Finalize with no agent activity before it took the lock on `main`.
The answer decides whether the landing queue note `landing-queue` is still worth building.

Until `finalize-owed-tips-gate` (`5c985d926`, 2026-11-28), a landing on a parent that was ahead of origin only by owed landing tips
lost all of its `landing.finalize_lost_races` races and then merged and ran verify under the parent lock.
Figures from landings before that commit can therefore include this fault.

## What to measure

From the drive reports and gate run logs of landings already made, no new runs:

- For each landing, the time from Finalize's start to taking the parent lock, the time holding it, and the lost races (`landing.finalize_lost_races`).
- How often a run fell back to merging and gating under the lock.
- The same figures for the landings before `d87a9ce24`, for comparison.

Cap: the last 20 landings before `d87a9ce24` and the first 20 after `5c985d926`, on the prime of this hub.
The report lands as a commit under `docs/research/`.
