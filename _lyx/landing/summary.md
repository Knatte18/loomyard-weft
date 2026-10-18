# Report darn runs beside loom runs in timing and convergence

`lyx report timing` and `lyx report convergence` left out darn runs.
Timing dropped every darn tag and in-flight darn pair, and convergence listed each darn tag as a partial run.
Both reports now select darn runs as they select loom runs, tag each run with its recipe, and time darn runs under phases of their own.

## What changes

- Run selection in `internal/reportengine` accepts a seed naming loom or darn, for landed runs (archive tags) and runs in flight (pair records branches).
  A batten seed or any other recipe stays a partial run or is left out, as before.
- `RunRow.Recipe` and `TimingRun.Recipe` carry the seed's recipe, and both markdown run tables gain a recipe column.
- Timing gains two phases, `write` and `review`.
  For a darn run, Darn-Write and Describe count under `write`, the Darn-Review perch rows under `review`, Merge-In under `merge`, and Publish, PR-Align, PR-Gate, Finalize and Friction-Reflect under `publish-finalize`.
- Phase medians are computed per recipe, so a loom run never enters a darn median.
  `TimingReport.Phases` is now a map from recipe to that recipe's phases, and the markdown phase-median table gains a recipe column.
- `internal/reportengine/doc.go` and `docs/overview.md` describe the change.

## Design decisions

- Phases come from a closed row-to-phase table per recipe (`recipePhaseTables`).
  Loom's table is unchanged except that it no longer maps the `Darn` family, which loom never runs.
  A row no table names still counts under `other` with a finding, so a new recipe's rows show up instead of vanishing.
- Darn's table also maps the retired row names a darn status may still carry, through `loomrecipe.DarnRetiredRows`.
- A Run-Shed change's producer is looked up in the run's own recipe table.
- A recipe's phases are those its table names plus `setup`, `land`, `paused` and `other`.
- `Describe` counts under `write` for darn because it is darn's last writing step, as it closes `webster` in loom.
- The seed is read through `shedrun.ParseSeed`, and the recipe and row names come from the `shedrun` and `loomshed` constants.

## Behavior to know

- The JSON shape of `phases` in `lyx report timing` changes from phase-to-median to recipe-to-phase-to-median.
  No code in the repository reads it.
- Convergence's weekly figures now sum over loom and darn runs, so a week's means shift as darn runs enter it.

## Checks

Not run by hand: `lyx report timing --since <date>` and `lyx report convergence` against the real records repository, to confirm darn runs such as `gate-wait-cache-cap` appear with recipe darn.
