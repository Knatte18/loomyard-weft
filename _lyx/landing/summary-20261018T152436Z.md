# Report darn runs beside loom runs, with phases per recipe

`lyx report timing` and `lyx report convergence` left out every darn run: run selection admitted only a seed naming the loom recipe, so a darn tag was a partial run and a darn pair in flight was skipped.
Darn writes the same run directory, status file and Finalize row as loom, so selection now admits both recipes and every figure reads a darn run's records as it reads a loom run's.

## What changes

- `RunRow.Recipe` and `TimingRun.Recipe` carry the recipe the seed names, and the timing markdown's run table gains a Recipe column.
- Convergence's `Week.Recipes` counts a week's runs per recipe, rendered as a "Runs by recipe" column in the wakes table, since the weekly figures now sum over both recipes.
- Timing maps a run's child rows through a closed phase table per recipe in `internal/reportengine/timingspans.go`.
  Loom's table is unchanged except that it drops the `Darn` row family, which no loom run holds.
  Darn's maps Preflight to setup, Darn-Write, its retired name `Darn` and Describe to a new `write` phase, the three Darn-Review rows to a new `review` phase, Merge-In to merge, and Publish, PR-Align, PR-Gate, Finalize and Friction-Reflect to publish-finalize.
  Batten rows, Run-Shed changes, sleep and wait accounting work as before, a change's producer being looked up in the run's own recipe table.
- `TimingReport.Phases` is now keyed by recipe, then phase: a median covers the landed runs of one recipe, and a recipe's map holds only the phases its runs can hold.
  The markdown phase-median table gains a Recipe column.

## Design decisions

- Describe counts under `write` in a darn run: it rewrites the writer's description from the final diff and closes darn's writing work, as it closes webster in loom.
- The phase set is a closed table per recipe rather than derived from the recipe file: the recipe's `segment` labels name review segments, not phases.
  A row a recipe gains before its table does still counts under `other` with a finding, so it shows up rather than vanishing.
- The JSON shape of `lyx report timing`'s `phases` changes from phase to median into recipe to phase to median; nothing in the repository reads it.

## Tests

Rows in `TestCompute_SelectsRuns` pin a finalized darn tag as a full run with recipe darn, an unfinalized one and a batten-seeded one as partial.
A `TestComputeTiming` row runs every darn row through its phase, puts a darn run in flight under `review` for its trailing interval, and checks that darn and loom medians stay apart.
The loom table's row now covers loom's rows alone, and the markdown tests cover both new columns.
