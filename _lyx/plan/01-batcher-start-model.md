# Card 1 — Fork start context grows with batch position

**Edit:**
- `internal/batcher/estimate.go`
- `internal/batcher/cost.go`
- `internal/batcher/config.go`
- `internal/batcher/template.yaml`
- `internal/batcher/doc.go`
- `internal/batcher/config_test.go`
- `internal/batcher/estimate_test.go`
- `internal/batcher/cost_test.go`
- `internal/websterengine/state_test.go`
- `internal/webstercli/cli_test.go`
- `tools/tokencount/calibrate.go`
- `tools/tokencount/calibrate_test.go`

**Intent:**
Replace the constant fork start with a start that depends on the batch's position (decision record D1, D3, D4, D6, D7).

Model, in `estimate.go`:
- `Weights` loses the `startup_context` coefficient and gains `MasterBase` (`json:"master_base"`) and `BatchGrowth` (`json:"batch_growth"`), both in the `coefficients` key list, so a missing or negative value fails as every other weight does.
- A profile whose `weights:` map still carries `startup_context` fails to load with an error naming `batcher.yaml`, the profile, `startup_context` and its two replacements `master_base` and `batch_growth`, in the style of the retired `alone_above` error in `registry.go`; check it before the unknown-key check so the error is the specific one.
- An old `state.json` must still load, and webster decodes it strictly (`state.ReadJSON` disallows unknown fields), so `Weights` keeps a decode-only field for the retired key (`json:"startup_context,omitempty"`), outside `coefficients`, documented as read only so a breakdown recorded before this change decodes; no new code writes it.
- `PeakContext` takes a 1-based `position` after the weights: the start is `MasterBase + (position−1) × BatchGrowth`, then `ForkMessages × MessageContext`, the read union and the write allowances, unchanged.
  `peakAccumulator.start` takes the position the same way.
- `Breakdown` gains `Position int` (`json:"position"`); `Startup` is the position-dependent start plus the startup messages; the weights it records carry `master_base` and `batch_growth`.
  `breakdownOf` takes the position.

Batchifier, in `cost.go`: the exact dynamic program runs over (batches so far, last card placed), since a segment's peak now depends on the position of the batch it forms; the k-th batch of the returned list is at position k (card 2 adds the handed-over offset).
A segment's position-independent part can be accumulated once per (begin, end) and the position term added per batch count.
Objective and tie-breaks are unchanged: fewest batches, then the lowest largest peak, then the last batch shortest.
Feasibility is unchanged: a one-card segment always; two or more cards need at most `MaxCards` and a peak at their own position within `Budget`.
Bound to state in the comments: `Budget` limits only segments of two or more cards, so a card whose own peak at its position exceeds it still runs alone, and nothing halts or warns; the recorded `Estimate` and `Position` show it.
Each returned `Batch` records its Estimate and Breakdown at its own position.

`template.yaml`: the `cautious` profile replaces `startup_context: 60000` with `master_base: 52000` (Merriam's context when it spawns its first fork) and `batch_growth: 7000` (context Merriam gains per finished batch); all other values stay.
`config.go`: `Active`'s comment lists the retired `startup_context` beside `alone_above`.
`doc.go`: rewrite the PeakContext and cost-batchifier paragraphs for the position model and the dynamic program's O(cards² × MaxCards) cost, and the fit paragraph per D7 — 52000 is the mean measured first-fork start (50.3K–53.5K) and 7000 the mean growth per batch (2.5K–13.0K) over three `cautious` runs and twelve forks with each start measured, one fork in a resumed Merriam session; growth counts batches, not cards (D3); the in-fork coefficients were fitted under the old constant start and await a re-fit.

Callers that must compile in this commit: `tools/tokencount/calibrate.go` calls `PeakContext` with position 1 for its one-card rows (card 5 replaces this); `calibrate_test.go`'s `seedProfile` writes `master_base: 10` and `batch_growth: 0` in place of `startup_context: 10`, so its arithmetic is unchanged; `internal/webstercli/cli_test.go`'s `batcher.Weights{StartupContext: 1000}` literal moves to `MasterBase`.

Tests, as rows of the existing tests:
- `config_test.go`: a cost profile with `master_base` and `batch_growth` loads; one with `startup_context` fails naming `batcher.yaml`, the profile, `startup_context`, `master_base` and `batch_growth`; a missing or negative `master_base` or `batch_growth` fails; the template's `cautious` profile carries 52000 and 7000 and no `startup_context`.
- `estimate_test.go`: `PeakContext` of a segment at position k equals the same segment at position 1 plus `(k−1) × BatchGrowth`.
- `cost_test.go`: the brute-force optimality check prices each batch at its own position; every multi-card batch stays within `MaxCards` and `Budget` at its position, every Breakdown records its position, and a card whose peak at a late position exceeds `Budget` comes out as a one-card batch with that over-budget Estimate.
- `internal/websterengine/state_test.go`: a `state.json` whose partition breakdown carries the old `weights.startup_context` key loads (extend the existing load test with that row).

**ImpactSummary:** batcher's estimator, cost batchifier, config template and doc change model; tokencount and two test files only follow the renamed weight and new parameter.
