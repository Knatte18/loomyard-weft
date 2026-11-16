# gate redundant: confirm each redundancy candidate by mutation

`lym gate redundant` listed a test as redundant when other tests executed all its blocks.
Executing a block is not checking it: two tests can run the same code and assert different results.
Every candidate the redundancy survey sampled in the five largest packages asserted something its covering tests did not.
The verb now keeps coverage as the cheap first filter and confirms each candidate by mutation before it lists it.

## What changes

- `internal/testcover` gains a mutator over `go/ast`.
  It takes the sites in a candidate's blocks and changes each by one operator:
  it negates an if or for condition or a comparison, drops a call, assignment or increment statement, or changes an integer, rune or string constant.
- Every site is wrapped in a helper that changes its value only while `LYM_TESTCOVER_MUTANT` names the mutant.
  The changed files go to `go test -c -overlay`, so one build holds every mutant and the worktree stays untouched.
  A mutant that does not compile, such as a constant a typed context needs, is dropped and the build runs again.
- `lym gate redundant` runs the candidate and its covering tests against each mutant.
  A candidate stays listed only when its covering tests kill every mutant it kills.
- The verb reports the candidates it refuted and those it did not judge, with the reason:
  a check that reads source files (marked `//lym:guard`, or reaching `internal/testkit/scankit`), a candidate that kills no mutant, or blocks that hold no mutant.
- A candidate's covering tests are picked from every other judged test, largest overlap first, leaving out tests whose check reads source files.
  The pick does not depend on which tests the coverage pass's greedy removal keeps.
- `internal/gatecli`, `internal/gateslot`, the README, `docs/overview.md` and `pattern/PATTERN-test-speed.md` describe the confirmed verb.

## Design decisions

- The mutator is lym's own, not go-mutesting or gremlins.
  The check needs a kill matrix (which test fails with which mutant), which those tools do not give without a build and a run per mutant, plus a new dependency.
- The covering set is a pick, not every test that shares a block.
  With the wider pool, `TestEnsure_PairDirectoryDeletedIsSkippedAsRemoved` and `TestStart_NeverAttaches` came back confirmed, since some unrelated test killed each of their mutants.
- A confirmed candidate is evidence, not proof: the operators cannot reach every check, such as one on another package's result.
- A candidate's kill beside other runs is not rerun alone, so a run another one disturbs can refute it falsely.
  A covering test's kill is rerun alone, and that run stands.
- The verb runs on command per package inside one gate slot, as `lym gate testmap` does, not on every gate.

## Measured

`docs/research/speed/redundancy-mutation-check-2026-11-16/report.md` holds the measurement, with raw output in its `data/`.
On the survey's five packages, 120 candidates gave 37 confirmed, 65 refuted and 18 not judged, in 13.3 minutes of wall (3 s for `internal/gitkit`, 8.6 minutes for `internal/loomcli`).
All six candidates the survey named are dropped: three refuted by a mutant their covering tests miss, two source scans, and one with no mutant in its blocks.

## To check by hand

- `redundant-test-waits-real-timeouts` (blocking, open): the redundant verb's tests wait out real timeouts, as the cost run suggests for `internal/loomcli`, whose runs mostly waited.
  Check that the test bounds in `internal/gatecli/redundant_integration_test.go` and the mutant run bound in `internal/gatecli/redundantmutants.go` do not slow the gate.
- The cost run did not record which `internal/loomcli` runs timed out; its 518 s wall on 293 CPU-seconds is the likeliest sign of mutants that hang a test until its bound.
- No regression check ran, and the cost figures are one run per package on a shared machine.
