# Finalize reruns a test failure before a fixer; reports name VSTest tests

A beam's Finalize landing spawned the merge fixer on one tier0 failure.
The failure came from a race between two tests, and a rerun would have passed.
The gate's failure report also said that no failing test or error line was read, since the tier was a .NET one, so the fixer started without a named cause.

Closes #100

## What changes

- **Finalize reruns a test-run failure once on the same head.**
  Finalize's landing gate now runs a failure once more before it judges it, when the failure report lists tier and guard runs alone.
  The rerun runs the failed tests alone, through `gateslot.FailedTestsCall`, when the report names Go tests only.
  Otherwise it runs the whole gate call again.
  A rerun that passes takes the first run's failures as flaky: the gate passes and no fixer spawns.
  The landing commit and the GitHub merge then list each such failure under `## Flaky test failures`, after the regression check.
  A rerun that fails is the gate's failure, and the landing failure record and the fixer read the rerun's log.
  A timeout, a runner crash, a shell that could not start and a build, vet or lint failure are not rerun.
  The gate of the fixer's own turns reruns the same way.
- **`gateslot.TierRunFailures`** reads a failure report's failures, each opened by its run's label, such as `tier0 example.com/m/a TestA`.
  It reports false unless every run the report lists is a tier or guard run, which is what decides a rerun.
- **The failure report names failed VSTest tests.**
  The `vstest` counter now reads each `Failed <name> [<duration>]` block of the VSTest console output.
  It keeps the test with the first lines of its block, which hold the error message and the first stack frames, without the section headings.
  The report names such a test by its name alone, since VSTest prints no package, and the run record carries it in `failures`.
- The final-summary spec names the new section among what Finalize appends.

## Design decisions

- Parent-Align's gate runs no rerun.
  The task asks for the rerun before Finalize's fixer, and Parent-Align lands nothing that could carry a flaky record.
- The flaky record is a section of the landing commit and the GitHub merge, not a file in the worktree.
  Finalize composes the landing texts after its gate, and a file written there would dirty the tree its clean-tree check reads.
- The flaky failures live in the Finalize process.
  A Finalize resumed after a flaky pass lists none that the earlier process found.
- A failure that names no Go test, such as a VSTest one, reruns the whole gate call.
  `FailedTestsCall` renders a Go `-run` pattern, which a VSTest tier cannot take.
- A rerun that passes the failed tests alone records that call's pass, not the full gate call's.
  A later Finalize pass on the same tree runs the full call again rather than skipping it.

## To check by hand

No regression check ran for this task, and none of the following was run against a live landing:

- A Finalize whose landing gate fails one test that passes on its rerun lands with no fixer, and its landing commit carries `## Flaky test failures` naming the test.
- A .NET tier's failing test shows in the gate's failure report with its exception message.
