# Fixer-report, round 1

## Changes

- F1: `internal/reportengine/doc.go:131` now reads "A darn run's setup also holds Preflight, Drift-Baseline and Drift-Await;", as proposed. Commit `f51a768f`.
- F2: the `contracts/recipes/darn-recipe.yaml` header now says a run that passed Drift-Baseline before Drift-Await joined the recipe has no `awaited_at`, halts at Drift-Verify with no counted baseline, and is settled by hand with `lym drift waive`. Fixed as proposed, no code added. Commit `4a7e3aca`.

## Self-check

- fix-proposal, MEDIUM, F1 doc sentence: disputed, premise false. The fork found no `darnPhases` because its grep missed the `loomshed.Name…` constants; `internal/reportengine/timingspans.go:106` maps `loomshed.NameDriftAwait` to `PhaseSetup`, so doc and table agree.
- fix-proposal, LOW observation on F2 (a run still capturing in Drift-Baseline is not covered): not a finding by its own account; no change.
- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

- Self-check F1 (fix-proposal), ground `premise false`: `internal/reportengine/timingspans.go:80` registers `darnPhases()` for `shedrun.RecipeDarn`, and line 106 reads `loomshed.NameDriftAwait: PhaseSetup`.

## Deferred

Nothing deferred.
