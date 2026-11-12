# Build the full set of coverage maps and list, per package, the tests the others already cover

`test-redundancy-candidates` · Task · enhancement, testkit, gate · draft

## Idea (operator, 2026-11-11)

The coverage maps exist to cut the test suite, which has far too many tests for this repository.
This task uses them: it builds the full set of maps and lists, per package, the tests the other tests already cover.
It cuts nothing; a later task cuts from the list.

## What it does

1. Builds a map for every package that has tier 0 or tier 1 tests, by the upkeep pass or a hand call of `lym gate testmap` naming the packages, whichever finishes the set.
   The build runs once per package; a package whose build fails is listed with its failure and skipped.
2. Produces, per package, a candidate list: every test whose covered blocks the package's other tests already cover, with its run time.
   It uses the judgment of the gate's redundancy step (`internal/testcover/verdict.go`) over the existing tests of a package, not only a diff's new ones.
   Tests marked `//testtiming:keep` and spawn-marked tests are neither candidates nor covers.
3. Writes a report under `docs/research/speed/` with the packages ranked by the run time their candidates would save, slowest first,
   the total candidate count and run time, and the count of packages with no map and why.

If no verb gives the candidate list today, the task adds the smallest one for it, with its tests.
It reads no test's assertions: equal coverage does not mean equal checks, which the cut task checks, one package at a time.

## Cap

One full set of maps, one candidate list per package and one report; no repeat runs.
`internal/fabricengine` is listed last and flagged: it was swept by hand on 2026-11-08 and is being reworked.

## Not in scope

Cutting or folding any test, and any change to the maps' format or upkeep rules.
