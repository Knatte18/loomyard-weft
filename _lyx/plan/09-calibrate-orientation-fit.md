# Card 9 — calibrate-orientation-fit

**Edit:**
- `tools/tokencount#StartRow`
- `tools/tokencount#StartFit`
- `tools/tokencount#StartFit.Unusable`
- `tools/tokencount#StartFit.write`
- `tools/tokencount#Calibrate`
- `tools/tokencount#startEstimate` -> `func startEstimate(w batcher.Weights, base batcher.StartBase, position int) float64`
- `tools/tokencount#fitStart`
- `tools/tokencount#Calibration.addStarts` -> `func (c *Calibration) addStarts(run RunTally, history PlanHistory, base BaseTrees) error`
- `tools/tokencount#Calibration.addRun`
- `tools/tokencount#Calibration.WriteMarkdown`
- `tools/tokencount/main.go`
- `tools/tokencount/calibrate_test.go`
- `tools/tokencount/calibrate_integration_test.go`

**Uses:**
- `plan:internal/websterengine#MerriamBaseOf`
- `tools/tokencount#readRunPlan`
- `tools/tokencount#BaseTrees`

**Intent:** The calibrate report fits `orientation` against each fork's measured start minus its run's computed Merriam base, the same base the live batchifier now prices with (D13's fit half).
For a run whose plan and base commit are readable, its computed base is `websterengine.MerriamBaseOf` over `CLAUDE.md`, `PATTERN.md` and the repository's copy of the Master template at the run's base commit (read through `BaseTrees`), plus the run's `00-overview.md` from its plan commit, priced at the profile's `context_per_line`.
`StartRow` gains that base and whether the run has one; its estimate is `base + orientation + (position − 1) × batch_growth`, which `startEstimate` computes from the base it now takes.
`addStarts` now reads each run's plan and base files, so it takes the plan history and base trees `Calibrate` already holds and returns a failed read as an error, as `addRun` does; a run whose base cannot be reconstructed is a row without a base, never an error.
`fitStart` fits `Orientation` and `BatchGrowth` by least squares of `measured − base` over `position − 1`, over the rows that carry a base; rows of a run without one stay listed with that reason and outside the fit, and fewer than two distinct positions among the fitted rows keeps today's "not fitted" answer.
`StartFit` reports `Orientation` beside `BatchGrowth`, `Unusable` checks both, and the markdown `StartFit.write` prints names `orientation` (profile value beside it) instead of `master_base`; `main.go`'s doc block is reworded to match.
`addRun` prices its peak estimates with the run's computed base, so the peak table and the start table agree.
`template.yaml`'s `orientation` stays 31400: no measured runs are read in this task, since a card never reads live files of other worktrees or the fabric repository; the report is how an operator later refits it.
`calibrate_test.go` updates the seeded profile and its fixtures so the fitted orientation is the seeded start minus a known computed base, and adds a row for a run without a readable base; the integration fixture gains the base files its tree needs.

**ImpactSummary:** Changes the calibrate report's start model and printed coefficient; the tool stays read-only and nothing consumes its fit.
