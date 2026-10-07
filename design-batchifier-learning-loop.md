# Fit the batchifier from logged runs: estimate vs actual peak context and cost

`batchifier-learning-loop` · Note · enhancement, webster

Operator, 2026-10-07: the batchifier's model is a first guess; treat tuning as a learning loop rather than a design-time decision.

- Log: each batch records its profile, coefficients and per-card estimate components in webster's `state.json` (being added with the peak-context budget change); the fork transcripts give the actual peak context (max input+cache tokens of any message) and the actual weight.
- Analyse: extend `tools/tokencount -calibrate` to read those records from finished runs and fit the peak-context coefficients (estimate vs actual peak) and the cost side (weight per batch vs batch size).
- Decide on data: the batch size and budget that minimise weight per card without a rise in failed batches, recoveries or Webster-Review findings; quality outcomes (batch failed, recover-batch, review findings per card) belong in the same table.
- Re-fit as runs accumulate, and compare profiles across runs before changing the default.