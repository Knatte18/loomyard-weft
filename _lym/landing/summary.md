# Batcher budget trial: recheck of profile wide against cautious

Adds `docs/research/batch-budget-trial-2026-11-01/`, the recheck the trial's follow-up asked for, with a recommendation on the two open questions.
No code or config changes.

## What changed

- `compare.sh`: the 2026-10-24 script, now reading both archive layouts (`_lyx/` and `_lym/`), the newer `batches` object in `state.json` and the review rounds under `reviews/code/`.
  Without this it found no runs after 2026-10-26.
- `README.md`: method, run and fork tables, findings and the recommendation.
- `runs.tsv`, `forks.tsv`: the script's outputs for this run.

## Result

- 23 `wide` runs against 59 `cautious`.
  No recoveries, no batches left not done, and one review round at the median.
- Cost per card is 1.13 for `wide` (9 runs with transcripts) against 1.15 for `cautious` and 1.495 for `cautious` runs of 10 cards or more.
- Across 29 `wide` forks estimated above 200k, the median peak is 0.89 of the estimate and the 90th percentile 1.27; 4 forks peaked over 300k, the highest at 381k.

## Recommendation

- `wide` goes into `internal/batcher/template.yaml` as the active profile, with `cautious` kept beside it.
- The estimator needs no global scale change above 200k: its error is spread, not bias.
  Lone large cards run about 40% under their estimate and multi-card forks reach 1.3 times over.

## Design decisions

- The `cautious` measured figures come from the 2026-10-24 baseline, because the transcripts of those runs are no longer on this machine.
  Only `wide` was measured in this run, and 15 of its 23 runs have no measured peak.
- The records clone was fetched (`git fetch --tags`) to include the latest archive tags; the repos were otherwise only read.
