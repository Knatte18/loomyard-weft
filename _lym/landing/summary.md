# tokencount counts darn runs and compares recipes by cost

`tools/tokencount` with no slugs counted only finished tapestry runs, so it skipped most recent runs, most of them darns, and no report put a darn's cost beside a tapestry's.
It now counts finished darn runs too and adds a recipe comparison.

## What changes

- With no slugs, tokencount counts finished runs of the tapestry and the darn recipe, a run being finished when its archived seed names one of them and its status holds a Finalize entry.
  Slugs named on the command line are still counted as given.
- `-since YYYY-MM-DD` counts every finished run whose sessions changed on or after the start of that local day, whatever `-last` says, so both recipes over one period stand side by side.
  `-last` below 1 is refused unless `-since` or slugs are given.
- New segments: Darn, Drift and Code-Review hold the roles of those names and their `-` suffixes, and Landing holds the `describe`, `pr-reviewer`, `fixer` and `fix-reviewer` roles, each a part labelled by its own name.
- "Cost per run" gains a recipe column, and its segment columns are the segments some counted run holds.
- A new "Recipes" section gives one row per recipe: runs, cost per run, span per run, runs that landed a commit, changed lines per landed run, and cost per changed line.
  A run whose archived seed was not read, such as one in flight, is listed under `unknown`.
  The section names each run without a landed commit and why.
- `-code` names the code repository read for landed commits and for the calibration's base trees, default the current directory.

## Design decisions

- A run's landed commit is found in `tools/tokencount/landed.go`: the newest commit whose subject is the title of the change description archived at the run's tag (`_lym/landing/summary.md`) and whose committer date lies within the run's span widened by a minute each way.
  The slack covers a pull request merged on GitHub, whose server clock stamps the commit.
  This assumes the landing default of one squashed commit titled after the change description.
- Changed lines are the lines added plus deleted against the commit's first parent, binary files left out; a root commit counts every line it holds.
- Cost per changed line divides the cost of the runs that landed a commit by the lines those commits changed, so runs with no landed commit do not inflate it.
- `internal/summaryparser` gains `ParseData`, which validates change-description content read from an archived revision; `Parse` now reads the file and delegates to it.
  `contracts/specs/loom-final-summary-spec.md` and the package doc say the parser takes content a caller read elsewhere.

## To check by hand

- Run `tools/tokencount.sh -since <date>` against a real hub with both darn and tapestry runs, and check the Recipes section's landed-commit matches against the code repository's history; the matching is covered by unit and fixture tests, not run against live archives.
- The `-since` path in `recentRuns`: check it counts the runs on and after the start of the given local day and no others, and that `-last` has no effect when it is given.
- A run whose archived change description does not parse: `findLanding` records it as having no landed commit with the parse reason, so the report lists the run instead of aborting; check that this holds for a real archive.
