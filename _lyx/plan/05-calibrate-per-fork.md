# Card 5 — Calibrate the start fit and every fork's peak

**Edit:**
- `tools/tokencount/calibrate.go`
- `tools/tokencount/calibrate_test.go`
- `tools/tokencount/calibrate_integration_test.go`
- `tools/tokencount/main.go`

**Intent:**
Implement D9 and D10 in the calibration section.

Start context (D9): for every fork of every counted run whose prompt names cards, a row with its run, its Merriam session, its position (the card-naming forks before it in the same session, by start time, plus one), its measured `StartContext` and the start the profile estimates for that position, `MasterBase + (position−1) × BatchGrowth`.
These rows need only the transcripts, so a run left out for its plan or base still contributes them; a resumed Merriam session restarts its positions at 1, and the webster records are not consulted.
Fit `MasterBase` and `BatchGrowth` by least squares of measured start over `position − 1` across all those forks, and print the fitted pair beside the profile's values with its residual spread, the 75th over the 25th percentile of the forks' measured start over the fitted start, interpolated as the existing `Fit` does.
With fewer than two distinct positions print the fit as not fitted with that reason, the rows still printed; a negative fitted coefficient is printed and marked unusable, since the loader refuses a negative weight.
The fit is a report and nothing consumes it.

Peak (D10): replace the one-card rows with one row per fork whose cards are all cards of the run's plan, one-card and multi-card alike: the estimate is `batcher.PeakContext` of the fork's cards at the fork's position over the run's base tree, beside the measured peak and its ratio.
Each row also carries the fork's own growth, measured peak minus measured start against estimated peak minus estimated start, and its ratio.
The fit lines report, per run and overall, the peak ratio's median and spread as now, and the in-fork growth ratio's median and spread.
The run-level skip reasons stay; the card-level "no fork names it" stays; the "ran in a multi-card fork" reason and its constant are retired; a fork naming a card the plan lacks is skipped with that reason; "estimate is 0" stays, per fork.
The read of a run's plan, which `Calibration.addRun` does inline (the newest plan commit before the run's first fork, parsed with `planparser.ParsePlanFrom` over the history), becomes one function that `addRun` and card 6's profile report both call.
`main.go`'s package comment describes the two tables, the position rule, the least-squares fit and its not-fitted and unusable marks, and the growth ratio, in place of the one-card description.

Tests:
- `calibrate_test.go`: fixtures with two webster sessions give positions restarting at 1 in the second; the least-squares pair comes out exact on starts laid on a line; a single position gives not fitted with its reason; starts falling with position give a negative growth marked unusable; a two-card fork gets a row whose estimate is `PeakContext` of both cards at its position, with its growth ratio; the per-run and overall growth medians are printed.
- `calibrate_integration_test.go`: the git-backed calibration still equals the in-memory one under the per-fork rows.
  Comments and strings in test files must not name the fabric side (`PATTERN-fabric-vocabulary`).

**ImpactSummary:** only tokencount's own calibration report format and tests change; no internal package or command consumes the section.
