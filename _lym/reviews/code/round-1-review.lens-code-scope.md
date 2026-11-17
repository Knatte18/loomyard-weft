```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points and listed the goal points no change builds.

**Decisions.** The record has only the board entry under `## Goal`; it has no `Added after Discussion` entries.
- **G1:** a run ended by its time bound or the memory cap is unjudged, not a kill. Those mutants are reported apart, and a candidate whose confirmation rests on such a run is listed as unconfirmed.
- **G2:** sort each test's `funcs` and `blocks` before the map is written (was B4).
- **G3:** record each test's time at a resolution that separates tests, or report that the figure is a floor (was B5).

**Per change:**
- **C1 → G1.** Unjudged runs are kept apart from kills. Covers:
  - `runMutants` sends a timed-out or memory-capped run to the `timedOut`/`capped` maps, never to `kills`, and returns `unjudged` (`internal/gatecli/redundantmutants.go:252-262`, `318-330`).
  - `JudgeMutants` takes `unjudged` and fills `MutantVerdict.Unjudged` (`internal/testcover/mutantverdict.go:67-107`).
  - `redundant.go` adds the `unconfirmed` list and the `mutantSites` helper (`internal/gatecli/redundant.go:58-70`, `285-296`).
- **C2 → G1.** A covering test that timed out beside other runs as a candidate is queued to run again alone (`redundantmutants.go:295-298`). Without this, a timeout caused by running side by side would leave a mutant unjudged that a solo run could decide.
- **C3 → G3.** `testOutput` stamps the `=== RUN` line and the result line, and `RunResult.Elapsed` gets their difference to the microsecond. `Toolchain.Now` is the injectable clock (`internal/testcover/runner.go:178-182`, `212-214`, `232-276`).
- **C4 → forced by C1/C3.** Doc and rule text describing the changed behaviour:
  - `docs/overview.md:966-967`
  - `internal/gatecli/doc.go:1198-1223`
  - `internal/testcover/doc.go`
  - `pattern/PATTERN-test-speed.md:315-319`
  - the help `Long` of `redundant.go`
- **C5 → forced by C1.** The help text has a length cap, so adding the unconfirmed sentence forced commit `8b745a81` to cut two existing clauses: the list of mutation operators, and the greedy pass's "while every block stays covered and every mutant killed". Information is lost, but the cut serves the traced change, so it is no scope finding.
- **C6 → forced.** Commit `0d1b952f` re-wraps comment lines only, in the files C1–C4 already touch.
- **C7 → forced.** Test changes for C1 and C3:
  - `TestJudgeMutants` rows
  - the `TestGateRedundant` expectations
  - the `TestRunAlone` row with the `standInTimed` stand-in
  - the `runMutants` call-site arity in `redundantmutants_guard_test.go:76`

**Per decision:**
- **G1:** built by C1 and C2.
- **G2:** no change, and the existing code already sorts both lists before writing:
  - `fillMapTests` sorts a new run's blocks with `slices.Sort` and its funcs with `slices.Compact(slices.Sorted(...))` (`internal/gatecli/testmappackage.go:650-651`).
  - A kept entry's lists go through `reindexed`, which sorts (`testmappackage.go:663-670`).
  - The function table is built with `slices.SortedFunc(..., CompareFuncKeys)` (`testmappackage.go:625`).
  - `WriteMap` hands the map straight to `state.WriteJSON` (`internal/testcover/coveragemap.go:182-185`).

  The goal's outcome already holds, so the missing change is no scope gap. The summary's account of this matches the code.
- **G3:** built by C3.

**Findings:** none. Every change traces to a goal point or is forced by one, and every goal point is either built or already met by existing code.

Tool calls:
1. `git show --stat 0d1b952f 8b745a81` and a grep for `WriteMap` in `internal/testcover/coveragemap.go`. My context lacked the file lists of the two fix commits, and the map writer's body, which I needed to settle G2.
