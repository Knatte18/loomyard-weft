# Decision record — timing-report-darn

## Goal

`lyx report timing` and `lyx report convergence` report darn runs beside loom runs, each run tagged with its recipe, so the two recipes can be compared from one report.

## Scope

- In: run selection in `internal/reportengine` (landed runs from archive tags and runs in flight on pair records branches) accepts a darn seed as it accepts a loom seed.
- In: a `recipe` field on each convergence run row and each timing run, and a recipe column in both markdown run tables.
- In: a per-recipe phase table for timing, with darn's own phases, and phase medians computed per recipe.
- In: the package docs of `internal/reportengine` and `internal/reportcli` describing the change.
- Out: any change to how darn or loom write their records, to the sleep and wait accounting itself, or to the board entry.

## Decisions

### Run selection accepts the loom and darn recipes

A tag or pair branch is a run when its seed names loom or darn; a batten seed, or any other recipe, stays a partial run or is left out as today.
Rationale: darn writes the same run directory, status and Finalize row as loom, so every read the selection makes applies to it unchanged, and convergence and timing share the selection, which fixes the gap in both.

### Each run carries its recipe

`RunRow.Recipe` and `TimingRun.Recipe` hold the seed's recipe name, and both markdown run tables render it as a column.
Rationale: weekly and bucket figures sum over runs of both recipes, so a reader tells them apart per run.

### Phases come from a closed table per recipe

Each recipe the selection admits has its own row-to-phase table.
Loom's stays as it is, less the `Darn` family it never runs.
Darn's maps Preflight to setup, Darn-Write and Describe to a new `write` phase, the Darn-Review perch rows to a new `review` phase, Merge-In to merge, and Publish, PR-Align, PR-Gate, Finalize and Friction-Reflect to publish-finalize.
Batten rows and Run-Shed changes count as today, with a change's producer looked up in the run's own recipe table.
Rationale: Describe is darn's last writing step, as it closes webster in loom, and the perch is darn's only review; a row no table names still counts under other with a finding, so a new recipe's rows show up rather than vanish.

### Phase medians are per recipe

`TimingReport.Phases` maps a recipe to that recipe's phases to their medians, and the markdown phase-median table gains a recipe column.
A recipe's phases are the ones its table names plus setup, land, paused and other.
Rationale: a median over loom and darn runs together describes neither, and comparing them is the reason darn runs enter the report.

## Constraints

- `PATTERN-told-geometry`: the engine stays told; the recipe comes from the seed it already reads.
- `PATTERN-sole-parsers`: the seed is read through `shedrun.ParseSeed`, the recipe names come from `shedrun`'s constants and the row names from `internal/loomshed`'s.
- `PATTERN-test-economy`: darn coverage goes in as rows of the existing selection and timing tests.

## Auto-mode assumptions

- Including darn runs in convergence is the fix the board's "checked for the same gap" asks for, since convergence lists every darn tag as partial today.
- Changing the JSON shape of `phases` in `lyx report timing` is acceptable: no code reads it.

## Open risks

- Convergence's weekly figures now sum over loom and darn runs, so a week's mean wakes shift as darn runs enter it.

## Acceptance criteria

- A darn tag with a Finalize entry is a full run in convergence, with recipe darn, and a loom tag keeps recipe loom.
- A landed darn run and a darn run in flight enter the timing report with recipe darn.
- A darn run's Darn-Write and Describe time counts under write, its perch rows under review, and Merge-In, Publish and Finalize under their shared phases.
- Phase medians are computed per recipe, a loom run never entering a darn median.
- A batten-seeded tag stays a partial run.
- The markdown renderings show the recipe of each run and of each median row.
