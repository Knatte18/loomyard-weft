# Trial: batcher budget 300k instead of 200k

`batch-budget-300k-trial` · Note · audit, webster

## Seen

`run-process-ownership` (2026-10-22) ran mostly one card per fork under budget 200k: its cards estimated 145-260k alone.
Measured fork peaks matched the estimates overall; `batch_growth` (7k) is above the ~2k measured when batches go well.
Many small forks were expensive under the identity batcher.

## Decision (operator, 2026-10-22)

The hub's `batcher.yaml` runs profile `cautious` with `budget: 300000` as a trial.
The hub layer pins the whole profile, since the config verb sets no deeper key, so later template changes to its weights do not reach this hub until it is removed.

## Follow-up

After a few loom runs, compare cost and fork quality against earlier runs; if good, move the budget into `internal/batcher/template.yaml` and drop the hub override.
