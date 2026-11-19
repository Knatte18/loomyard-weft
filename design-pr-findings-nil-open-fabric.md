# pr-findings fails: NewFinalize Deps.OpenFabric must not be nil

`pr-findings-nil-open-fabric` · Task · bug, loom, landing · running · Code-Bouncer

Imported from [issue #155](https://github.com/Knatte18/loomyard/issues/155).

## What was seen

In a darn run's PR-Review row, `lym loom pr-findings` failed twice on a findings file that passes the burler format check, with: `loomrecipe: shedbuild: producer 13 "Finalize": landingshed: NewFinalize: Deps.OpenFabric must not be nil`.

## Workaround

The PR-Review agent fixes the parent's findings by hand and commits, without `pr-findings`.

## Expected

`pr-findings` builds the recipe's shed with its fabric dependency set and runs the findings round.