# Card 2 — batcher-orientation

**Create:**
- `plan:internal/batcher#StartBase` -> `type StartBase struct`
- `plan:internal/batcher#ErrRetiredKey` -> `var ErrRetiredKey error`

**Edit:**
- `internal/batcher#Weights`
- `internal/batcher#coefficients`
- `internal/batcher#profileWeights`
- `internal/batcher#newCostFromProfile`
- `internal/batcher#Active`
- `internal/batcher#loadConfig`
- `internal/batcher#Batcher.Batch`
- `internal/batcher#costBatcher.Batch` -> `func (b costBatcher) Batch(plan *planparser.Plan, cards []planparser.Card, sizes SizeSource, before int, base StartBase) ([]Batch, error)`
- `internal/batcher#identityBatcher.Batch` -> `func (identityBatcher) Batch(_ *planparser.Plan, cards []planparser.Card, _ SizeSource, _ int, _ StartBase) ([]Batch, error)`
- `internal/batcher#PeakContext` -> `func PeakContext(plan *planparser.Plan, cards []planparser.Card, sizes SizeSource, w Weights, base StartBase, position int) (float64, error)`
- `internal/batcher#breakdownOf` -> `func breakdownOf(cards []planparser.Card, loads []cardLoad, w Weights, base StartBase, position int) Breakdown`
- `internal/batcher#startContext` -> `func startContext(w Weights, base StartBase, position int) float64`
- `internal/batcher#peakAccumulator.start` -> `func (p *peakAccumulator) start(w Weights, base StartBase, position int)`
- `internal/batcher/template.yaml`
- `internal/batcher/doc.go`
- `internal/batcher/config_test.go`
- `internal/batcher/cost_test.go`
- `internal/batcher/estimate_test.go`
- `internal/batcher/identity_test.go`
- `internal/websterengine#batchCards`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/partition_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/state_test.go`
- `internal/webstercli/cli_test.go`
- `tools/tokencount#startEstimate`
- `tools/tokencount#Calibration.addRun`
- `tools/tokencount#StartFit.write`
- `tools/tokencount/calibrate_test.go`

**Uses:**
- `plan:internal/configengine#MarkInvalid`

**Intent:** Replace the fitted `master_base` with a computed base plus an `orientation` weight in the cost model (D13's batcher half), and retire the old keys loudly (D12's loader half).

The start model:
- `StartBase` is the part of Merriam's start context computed from what Merriam loads: `Lines int` (the loaded text's line count) and `Fixed float64` (tokens for the system prompt and tools).
  Its context under weights `w` is `Lines × w.ContextPerLine + Fixed`; the zero value adds nothing.
- `Weights.MasterBase` becomes `Weights.Orientation`, JSON key `orientation`, the coefficient key `orientation` in `coefficients`.
  A read-only `RetiredMasterBase` field with JSON key `master_base,omitempty` lets a breakdown recorded in an older `state.json` decode, exactly as `RetiredStartupContext` does; it is outside `coefficients` and nothing writes it.
- `startContext` takes the base: `base context + Orientation + (position − 1) × BatchGrowth`.
- `Batcher.Batch` gains a `base StartBase` parameter after `before`; `costBatcher.Batch` prices every segment with it and the identity batchifier ignores it, as it ignores `before`.
  `PeakContext` gains the same parameter before `position`.
- `breakdownOf` passes the base to `startContext`, so `Breakdown.Startup` includes it; `Breakdown` gains no field, since the calibrate fit (card 9) recomputes each run's base from its plan and base commit and a recorded base would have no reader.

The retired keys: `profileWeights` refuses `weights.master_base` and `weights.startup_context`, and `newCostFromProfile` refuses the profile-level `alone_above`.
Each refusal wraps `ErrRetiredKey`, names `batcher.yaml`, the profile and the key, and ends `way forward: run "lyx config reconcile --apply"`, which card 4 makes perform the migration.
Every validation failure of `Active`, `loadConfig`, `newCostFromProfile` and `profileWeights` (unmarshal, an active naming no profile, an unknown batchifier kind, a missing or invalid parameter, an unknown or retired weights key) is wrapped with `MarkInvalid`, so card 6 can tell config content from an unreadable file.

`template.yaml` replaces `master_base: 52000` with `orientation: 31400`, the measured 52000 first-fork start minus the measured 20.6K Merriam session start, and rewords the `batch_growth` comment to the new start formula.
The package doc states the new start model, where the base comes from (webster computes it, card 8), the 31400 derivation and the retired keys with their reconcile way forward.

Callers stay compiling with no behavior of their own yet: `batchCards` passes `batcher.StartBase{}` until card 8 computes the real base, and `startEstimate` and `Calibration.addRun` in `tools/tokencount` read `Orientation` and pass `batcher.StartBase{}` until card 9 refits, and `StartFit.write` prints the profile's `Orientation` where it printed `profile.MasterBase`, its label unchanged until card 9 rewords it.
Grep the tree for `master_base`, `MasterBase` and the old `startup_context` refusal text; every test asserting them joins this card: the batcher tests (their `Batch`/`PeakContext` calls and the weights fixtures), the webster test fakes implementing `Batch` and the `Weights` literals in `rebaseline_test.go` and `cli_test.go`, and the `calibrate_test.go` seeded profile.
`config_test.go` gains rows for a profile carrying `master_base` and one carrying `alone_above`, each asserting `errors.Is(err, ErrRetiredKey)`, `errors.Is(err, configengine.ErrInvalid)` and the reconcile way forward, and the template row asserts `orientation` 31400.
`estimate_test.go` pins that a non-zero `StartBase` raises `PeakContext` by exactly its context.
`state_test.go` gains a row decoding a breakdown recorded with `master_base`.

**ImpactSummary:** Changes the Batcher interface and PeakContext signature for every implementation and caller, and makes a batcher.yaml still carrying master_base fail to load until reconciled.
