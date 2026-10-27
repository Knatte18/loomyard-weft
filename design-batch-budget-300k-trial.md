# Trial: batcher profile wide (budget 300k)

`batch-budget-300k-trial` · Note · research, webster

## Seen

`run-process-ownership` (2026-10-22) ran mostly one card per fork under the `cautious` profile's budget 200k: its cards estimated 145-260k alone.
Measured fork peaks matched the estimates overall; `batch_growth` (7k) is above the ~2k measured when batches go well.
Many small forks were expensive under the identity batcher.

## Decision (operator, 2026-10-22)

`cautious` stays as it is; the hub's `batcher.yaml` adds a profile `wide`, the same as `cautious` but with `budget: 300000`, and makes it `active`.

## Follow-up

After a few loom runs, compare cost and fork quality against earlier runs; then decide whether `wide` goes into `internal/batcher/template.yaml`.

## First check (Sonnet agent, 2026-10-24)

The hub switched at 2026-10-22T15:50 (records commit `216c22a12`); each run's profile comes from its `_lym/webster/state.json`.
Only 2 wide runs with a Webster record have landed (`orch-strand-cycle`, `loom-tail-review`, the latter run on another machine, so no cost data), against 51 cautious runs since 2026-10-15.

| median | cautious (51) | cautious, 10+ cards (18) | wide (2) |
|---|---|---|---|
| cards per fork | 3.0 | 2.1 | 3.1 |
| measured fork peak | 171k | 192k | 310k (1 run) |
| Webster cost per card | $1.15 | $1.50 | $1.12 (1 run) |
| review findings | 1 | 2 | 2 |

- Too few wide runs to decide; no quality regression seen (no stuck batches, one review round).
- In the one measured run, forks of 4-5 cards peaked at 310k and 315k, 31-39% over the batcher's estimate; cautious forks average 0.99 of estimate. If `wide` stays, the estimator needs calibrating above 200k.
- Recheck after 6-8 more wide runs. Rerun with `docs/research/batch-budget-trial-2026-10-24/compare.sh [FROM-date]`; its README and the 2026-10-24 baseline tables sit beside it.
