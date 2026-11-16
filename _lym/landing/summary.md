# lym gate redundant confirms each coverage candidate by mutation

`lym gate redundant` called a test redundant when the package's other tests ran all its blocks.
Running a line is not checking it, and the survey of 2026-11-16 found every sampled candidate asserting something its covering tests do not.
The verb now keeps coverage as the first filter and confirms each candidate by mutation before listing it.

## What changes

- **Mutation check.** For a package with coverage candidates, the verb takes one gate slot.
  It builds the package's coverage binary to place the candidates' blocks in the source, then one test binary holding every mutant of those blocks.
  A mutant negates an if or for condition or a comparison, drops a call, assignment or increment statement, or changes an integer, rune or string constant.
  Each candidate runs with each mutant of its blocks on; each covering test runs with each mutant a candidate killed, where its entry covers that block.
- **Verdict.** A candidate stays in `candidates` only when it kills a mutant and a covering test kills every mutant it kills, with `mutants_killed`.
  One that kills a mutant its covers miss goes to the new `refuted` list, with those mutants' file, line and operator.
  One that does not pass unmutated, holds no mutant or kills none goes to `not_judged` with the reason.
  `removable` now needs both the coverage pass and a kill pass, which removes a candidate only while a test it keeps kills each mutant the candidate killed.
- **Source checks.** A contained test marked `//lym:guard`, or one reaching `internal/testkit/scankit`, is not judged: its check reads source files no mutant changes.
- **New refusal.** The verb refuses a map built from other Go files of the package than the worktree holds, pointing at `lym gate testmap <package>`.
  With no contained test it still runs nothing and takes no slot.
- **Output.** A new `mutation` object reports the mutants built and dropped, build and run seconds and the run count.

## Where it lives

- `internal/testcover/mutant.go`: `PlanMutants`, `MutantPlan.WriteOverlay` and `UnbuildableMutants`, the mutator over `go/ast`.
  Every mutant sits behind a helper keyed by `LYM_TESTCOVER_MUTANT`, in one overlay for `go test -c -overlay`; no line moves, so compiler errors name the mutant.
- `internal/testcover/mutantverdict.go`: `JudgeMutants`, the pure kill-matrix verdict.
- `internal/testcover/sourcescan.go`: `ScanSourceReaders`.
- `internal/testcover/runner.go`: `BuildMutantBinary` with `MutantBuildError`, `ProfileBlocks`, and `RunAlone` with no profile for a binary built without coverage.
- `internal/gatecli/redundant.go` and `redundantmutants.go`: the verb and its slot-held check; `redundantCmd` now takes the go binary and timeout units.
- Docs: the `testcover`, `gatecli` and `gateslot` package docs, `docs/overview.md` and `pattern/PATTERN-test-speed.md`.

## Design decisions

- **Own mutator, one build per package.** Existing Go mutation tools build each mutant and report whether any test killed it.
  The check needs which test killed which mutant, so a schema with all mutants in one binary keeps builds at three per package and adds no dependency.
- **Unbuildable mutants are dropped, not predicted.** A constant behind the generic helper fails in a typed context such as `5 * time.Second`.
  The build retries up to `mutantBuildRounds` times, each time dropping the mutants on the lines the compiler names; 23 of 1,151 dropped in the measurement.
- **Covering tests are the set cover the coverage verdict picks** among every other judged test (`testcover.CoveringTests`), run only on mutants a candidate killed.
  The coverage pass's own `covering` list for a removable test comes from the tests its greedy pass kept, so it varies with the tests' seconds and could miss the test that kills the same mutant.
  Source-checking tests are no covers, since they kill no mutant.
  A wider pool of every test sharing a block confirmed two of the survey's sampled candidates, so the check keeps to the board entry's covering tests.
- **Mutant timeouts** are ten times the test's unmutated seconds, at least 10 s, within `redundancy_test_timeout_sec`, so an endless loop ends early; a timeout counts as a kill.
- **Scope.** The gate's `redundancy` report step and `cmd/testtiming -redundancy` still judge by coverage alone; their output stays evidence for review, as PATTERN-test-speed says.

## Measurement

`docs/research/speed/redundancy-mutation-check-2026-11-16/report.md` records the run on the survey's five packages.
All six candidates the survey named are dropped: three refuted, two source scans, one with no mutant in its blocks.
Of 120 coverage candidates, 37 stay listed, 65 are refuted and 18 are not judged.
The check cost 13.3 minutes of wall over the five, from 3 s for `internal/gitkit` to 8.6 minutes for `internal/loomcli`.

## Tests

- `TestPlanMutants`, `TestMutantPlan_WriteOverlay` and `TestMutantPlan_UnbuildableMutants` pin the sites, the overlay text and the drop mapping (tier 0).
- `TestJudgeMutants` pins the verdict and the kill-pass removable set; `TestScanSourceReaders_MarksTestsReachingTheScanKit` the scan;
  `TestCoveringTests_TakesTheLargestOverlapFirstAndSkipsARunAddingNothing` the covering pick (tier 0).
- `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` keeps one test in its stand-in module, so the verb reads the map with nothing to build.
- `TestBuildMutantBinary`, `TestProfileBlocks` and a no-profile `TestRunAlone` row pin the runner's commands against the stand-in (tier 1).
- `TestGateRedundant` now builds a real fixture module with `lym gate testmap` and the real go command.
  It checks a confirmed pair, a refuted test with its mutants, a test that kills nothing, a source scan, and the new stale-map refusal (tier 1).
