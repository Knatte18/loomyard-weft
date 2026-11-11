# gate: add a coverage scope that narrows importing packages' tests

A gate key can now hold `scope: coverage`, beside `card`, `diff` and `all`.
Under the `go` scoper it computes the `diff` scope, then narrows it:
a changed package runs every test of the tier, and a package in the set only because it imports a changed package runs
the tests its coverage map selects.
Under any other scoper, `coverage` computes the same targets as `diff`.
The point is to cut the tier 0 and tier 1 time of the Darn write gate, which carries most of the measured gate minutes.

The template's `darn-write`, `webster-fix` and `code-burler-fix` keys now default to `coverage`; `pr-review` stays `diff`.
The `finalize` landing gate keeps scope `all`, so a test a coverage run skipped is still caught before a landing.

## What changes

- `internal/gateslot`: `coverage` is a valid scope value, and the scope error names it.
  The loader stays strict, so a machine on an older binary refuses a hub file that holds `coverage` until it updates.
  The template comment and the package doc list the new value and which keys default to it.
- `internal/impactset`: `Targets` now separates `Importers` and `BinaryBuilders` from the packages the diff touches.
  The `diff` set itself is unchanged.
- `internal/gatecli/coveragescope.go` (new): the narrowing.
  An importing package runs, per tier, the union of:
  - the tests the map's selector selects;
  - tests of the tier the package declares that its map does not record;
  - tests its map marks out of process or unresolved;
  - its `//lym:guard` tests;
  - the whole package on any selector miss.
  The union holds only tests that a run of the tier executes: declared in a test file the host admits under the tier's tag,
  and not skipped by the diff's watch skip.
  A package with an empty union and no miss runs nothing for that tier.
- A tier runs its changed and whole-run packages as one invocation, as before.
  Its selected packages run as a second invocation with an anchored `-run '^(<names>)$'`.
  The filter sits in the run's own args, not the call's, so the empty-tier check still reads the run.
  The selected invocation's green entry carries the filter, so it never serves as a later gate's base
  and never stands in for a whole run.
  A pattern past `greenCommandBudget` runs the selected packages whole instead.
  `--stop-on-fail`, the failed-test rerun and the verify budget act on both invocations.
- The scope report names each package of a coverage-scoped tier:
  `changed`, `whole: lym binary`, `miss: <reason>`, `whole: over budget`, `selected <n> of <m>` or `none selected`.
- `internal/fabricengine`: `CodeWorktreeChanges` lists uncommitted and untracked paths of the code checkout,
  which the selector uses to compare the working tree with a map.
- `internal/testcover`: a small change in the selector and its doc for the new caller.
- `pattern/PATTERN-test-speed.md` records the new scope.

## Design decisions

- `coverage` only removes tests from packages that are not changed; it never adds a package `diff` would not run.
  Watch skips, the whole-tree fallbacks, `//lym:guard` and `inert_paths` act as under `diff`.
- A call with explicit targets is never narrowed: Darn's `tree_scope` targets, the failed-test rerun and a hand-named list.
  A call whose words after `--` carry a filter flag, and a tier with no `{args}` or target placeholder or with `carry: false`,
  are not narrowed either.
- The gate only reads maps, through `testcover.ReadMap` under the hub's `.lym/testcover`; it never starts a build or a refresh.
  An unreadable map, an unopenable selector or an unreadable working file is a miss, and the package runs whole.
  File content at a map's commit is read through `fabricengine`.
- Coverage narrows only while a map is fresh.
  The share of fresh maps is set by the map upkeep cadence (note `testmap-upkeep`), which this change does not touch.

## Known gaps

- A test that runs changed code in a child process the spawn scan does not see is mapped as in-process and not selected.
  The scan does not follow a call into another in-module package that is not a testkit,
  gives every test in a tier0 file an empty verdict,
  and does not see a hook or alias that makes git run module code.
  Only the landing gate catches such a test.
- A tier whose command builds or vets first does so in both invocations; the build cache serves the second.

## To check by hand

Nothing was run by hand beyond the gate.
These findings from review were fixed or disputed, and no fresh reviewer has seen the result; check each:

- `computeScope`'s coverage branch in `internal/gatecli/scope.go`: whether the tests pin it.
- The coverage selector setup in `internal/gatecli/coveragescope.go` (`openSelector`): whether the tests pin it.
- The tier table in `internal/gateslot/doc.go`: whether it states the scope correctly.
- The selected-test run in `internal/gatecli/tiers.go` (`selectedTestsRun`): whether the tests execute it.
- The recorded selection in `internal/gatecli/coveragescope_test.go`: whether the tests assert it.
