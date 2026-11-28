# Measure how much landing-lock-only-for-land cut the wait for the parent lock

`landing-lock-effect-measure` · Note · research, landing, batten

## Why

`landing-lock-only-for-land` (`d87a9ce24`) made Finalize merge and gate with no parent lock held, and take the lock only to land.
Whether ready runs now land without queuing is unmeasured.
On 2026-11-28 `reset-partial-commit-push` still showed 38 minutes at Finalize with no agent activity before it took the lock on `main`.
The answer was to decide whether the landing queue was still worth building.
The queue's Sketch `landing-queue` has since been split into the tasks `landing-ledger-own-tip` and `landing-queue-batch`,
and the latter replaces the lock-free gate this note measures.

Until `finalize-owed-tips-gate` (`5c985d926`, 2026-11-28), a landing on a parent that was ahead of origin only by owed landing tips
lost all of its `landing.finalize_lost_races` races and then merged and ran verify under the parent lock.
Figures from landings before that commit can therefore include this fault.

Since `db892e0f9` (2026-11-28), a gate call of a run that holds the parent lock takes a free gate slot ahead of other waiters,
so slot waits under the lock before and after that commit do not compare.

## What to measure

From the drive reports and gate run logs of landings already made, no new runs:

- For each landing, the time from Finalize's start to taking the parent lock, the time holding it, and the lost races (`landing.finalize_lost_races`).
- How often a run fell back to merging and gating under the lock.
- The same figures for the landings before `d87a9ce24`, for comparison.

Cap: the last 20 landings before `d87a9ce24` and the first 20 after `5c985d926`, on the prime of this hub.
The report lands as a commit under `docs/research/`.
