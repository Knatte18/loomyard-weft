# Report the gate trials that cut new test binaries per run

Adds `docs/research/gate-fewer-binaries-trial-2026-11-01/`, a report with its scripts and tables on two trials of the cuts in `docs/research/gate-fewer-launches-2026-10-31/report.md`.
No production or test code changes: each trial was applied to the working tree, measured with `lym gate test`, and reverted, and only its patch is kept in the directory.

## What the report finds

- One test binary per package for all tiers, with the tier chosen at run time, is the cut worth making permanent.
  The package graph gives 532 test binaries on a cold tree before and 182 after, and about 101 relinked by a change of mean fan-in before and about 35 after.
  On a ten-package sample the same tests passed before and after in every tier, with the other tier's tests skipped by a run-time selector.
- Cutting `lymtemp`, `lock`, `proc` and `agentname` out of the common `TestMain` path saves only when one of them changes (184 of 544 new binaries for `internal/agentname`), nothing in tier2, and breaks `TestRunDirectory_HoldsEveryTempFileAndIsGoneAfterTheRun`.
- Merging small leaf packages was not trialled.

## Design decisions

- Trial 2 ran on a sample, as the board entry says, and the whole-module figures come from `scripts/fanin-union.sh` over the package graph; the sample confirmed the graph's prediction for two of three perturbed packages, and the third differs by the noise floor.
- Trial 3 was left out because the entry made it conditional on trials 1 and 2 falling short, and trial 2 does not; the report says so and leaves the call to the operator.
- The count of new binaries is the number of packages in a tier record's `launches`, an upper bound that includes tests which read the prebuilt `lym`; the report subtracts nothing and gives the floor of an unchanged tree beside each trial.
- The trial patches are not applied anywhere in the tree, and trial 2's patch for the sample's test files is not stored: `scripts/untag.sh` regenerates it.

## What is left for the operator

The OSL-1046 check run the entry asks for is described in the report under "Check on OSL-1046"; no agent ran there.

## Testing

No code changed, so no test run applies.
The numbers come from the gate records of the trial runs, listed per run in each trial's `results.tsv`.
