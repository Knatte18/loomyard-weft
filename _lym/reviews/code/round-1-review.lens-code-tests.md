```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests under the code-tests lens. For each behaviour the diff adds or changes, name the test that pins it.

**Per behaviour, the test that pins it**

- **`PlanMutants` sites and operators:** pinned by `TestPlanMutants` and indirectly by `TestMutantPlan_WriteOverlay`. Its rows for a declaring assignment, a for post statement, a const declaration and a blank assignment exist only in `TestPlanMutants`; see F1.
- **`WriteOverlay` text, a dropped mutant left out, no line added:** pinned by `TestMutantPlan_WriteOverlay`.
- **`WriteOverlay` leaves a file with every mutant dropped unreplaced:** none (F8).
- **`applyMutants` order of nested insertions at one offset** (for example `if n < 0` with its literal `0`): no tier-0 row, because `WriteOverlay` drops mutant 2. `TestGateRedundant` pins it at tier 1 through `Mutants == 5`, `Unbuildable == 0`.
- **`UnbuildableMutants`:** pinned by `TestMutantPlan_UnbuildableMutants`. A mutant spanning several lines (the `lastLine` bound) has no row (F8).
- **`JudgeMutants`** (redundant, refuted, three not-judged reasons, ascending `Killed`, greedy removable): pinned by `TestJudgeMutants`.
- **`ScanSourceReaders`:** pinned by `TestScanSourceReaders_MarksTestsReachingTheScanKit`.
- **`sourceCheckingTests`, scankit branch:** pinned by `TestGateRedundant` (row `TestSourceScan`).
- **`sourceCheckingTests`, `//lym:guard` branch:** none (F3).
- **`CoveringTests`:** pinned by `TestCoveringTests_TakesTheLargestOverlapFirstAndSkipsARunAddingNothing`.
- **Covering pool in `mutationCheck.run`** (every judged test except source checks and unjudged tests, independent of the greedy pass): no deterministic row (F4).
- **`changedGoFiles` stale-map refusal, hash differs:** pinned by `TestGateRedundant`, row `RefusesAMapBuiltFromOtherFiles`. A file present on one side only has no row (F8).
- **No contained test: reads the map, builds nothing:** pinned by `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` (exit 0 and `map_commit`).
- **Refusals for the source scan, Go-file hashing, the env config, a failed mutation check and a cancelled call:** none (F2).
- **`buildMutants` drop-and-rebuild loop, its `mutantBuildRounds` cap and a refused line no mutant spans:** none (F5).
- **`mutantTimeout`** (factor, floor, cap at `redundancy_test_timeout_sec`) **and a timed-out run counting as a kill:** none (F6).
- **`runJobs` reruns a covering test's kill alone:** none (F8).
- **`runMutants` sets `FailsUnmutated` on a candidate failing its unmutated run:** only `JudgeMutants` with the flag preset; the gatecli path that sets it has no row (F8).
- **Candidate `removable` = coverage pass && kill pass:** pinned by `TestGateRedundant` ("at most 1 removable"). This holds only when the coverage pass happens to mark both removable, which depends on the run's seconds.
- **Refuted output with unique mutants' file, line and operator:** pinned by `TestGateRedundant`.
- **`mutation` stats:** `Mutants`, `Unbuildable` and `Runs > 0` are pinned by `TestGateRedundant`. The seconds fields are not asserted, which is acceptable.
- **`BuildMutantBinary` args and `MutantBuildError` lines:** pinned by `TestBuildMutantBinary`. A non-compile failure returning a plain error has no row (F8).
- **`ProfileBlocks` success and args:** pinned by `TestProfileBlocks`. Its failure, timeout and missing-profile paths have no row (F8).
- **`RunAlone` / `runAloneArgs` with no profile:** pinned by `TestRunAlone`, row `PassWithNoProfileFlag`.
- **Inherited slot in `holdSlot` (`gateslot.InheritEnv`):** none (F8).
- **`buildLymFor` with a `prebuilt_lym` tier:** none (F8).
- **Exit code 128 plus the signal number after a terminating signal:** none in this diff; the shared helper is used (F8).

**Findings**

- **F1 — MEDIUM, consistency.** Location: `internal/testcover/mutant_test.go:1765` (`TestPlanMutants`).
  - Summary: the redundancy report lists it as contained, covered by `TestMutantPlan_WriteOverlay`. Its assertions reach what that test cannot see: the rows for `blockCountHead` (a declaring assignment and a for post statement are not mutated) and `blockCountTail` (a const declaration and a blank assignment hold no mutant). `WriteOverlay` targets neither block.
  - Fix: accept it and mark it `//testtiming:keep pins the sites the mutator leaves alone: declaring and blank assignments, for post statements and const literals` directly above `func TestPlanMutants`.
  - Evidence: `mutant_test.go:1789-1801` against `1825`.
- **F2 — MEDIUM, design.** Location: `internal/gatecli/redundant.go:294-297`, `341-351`, `367-374`.
  - Summary: five new refusals have no test that reaches them and asserts the message: the source scan failed, cannot hash the Go files, cannot read the env config, the mutation check failed, and the call was cancelled. PATTERN-refusal-way-forward requires one, and `contracts/specs/refusal-spec.md` holds no exemption.
  - The cheapest to reach are a failed mutation check (a fixture package whose mutant build fails on a line no mutant spans, or a build timeout) and a failed source scan (a `.go` file that does not parse).
  - Evidence: `pattern/PATTERN-refusal-way-forward.md:9`; the refusal table in `redundant_integration_test.go` has only the stale-map row.
- **F3 — MEDIUM, design.** Location: `internal/gatecli/redundant.go:402-416` (`sourceCheckingTests`, the `impactset.FindGuardTests` branch).
  - Summary: no fixture test carries `//lym:guard`. Deleting the guard loop fails no test, though `doc.go`, `PATTERN-test-speed.md` and the summary all promise a guard test is never judged.
  - Evidence: the `redundantModuleFiles` fixture (`redundant_integration_test.go:492-567`) holds no guard marker.
- **F4 — MEDIUM, design.** Location: `internal/gatecli/redundantmutants.go:857-868` (the covering pool of `run`) and `redundant.go:302-308` (`covers`).
  - Summary: commit `e6a52ffc` makes the covering pool every judged test except source checks, independent of the greedy pass. No row pins it deterministically.
  - In the fixture, `TestSourceScan` and `TestKept` kill nothing. Removing them from `covers`, or adding them, never changes an asserted output.
  - Reverting to `verdict.Covering` fails `TestGateRedundant` only when the seconds order the greedy pass so that `TestDoubleChecked`'s covers come out as `[TestDoubleUnchecked]`. The test zeroes `Covering` because it varies by run.
  - Fix: add a deterministic row, such as one where the greedy-kept cover kills nothing but another judged test does.
  - Evidence: `redundant_integration_test.go:673-675`.
- **F5 — MEDIUM, design.** Location: `internal/gatecli/redundantmutants.go:1011-1035` (`buildMutants`).
  - Summary: the drop-and-rebuild loop is the design decision behind "unbuildable mutants are dropped, not predicted" (23 of 1,151 dropped in the measurement), yet no test exercises it.
  - The integration fixture builds with `Unbuildable == 0`. `UnbuildableMutants` is unit-tested alone, but neither the rebuild, the `mutantBuildRounds` cap nor the "refused line no mutant spans" exit is.
  - A fixture line such as `time.Duration(5) * time.Second` or `5 * time.Second` would reach it.
  - Evidence: `redundant_integration_test.go:683`.
- **F6 — LOW, design.** Location: `internal/gatecli/redundantmutants.go:979-986` (`mutantTimeout`) and the `record` closure at `926-935`.
  - Summary: the bound (ten times the unmutated seconds, at least `mutantTimeoutFloor`, capped by `testTimeout`) and "a timeout counts as a kill" have no row.
  - Evidence: no test references `mutantTimeout`, and the fixture holds no looping mutant.
- **F8 — LOW, design.** Uncovered secondary paths, one finding:
  - `runJobs` alone-rerun of a covering kill (`redundantmutants.go:1048-1064`).
  - `runMutants` setting `FailsUnmutated` (`939-941`).
  - `holdSlot` inside an inherited slot (`1138-1143`).
  - `buildLymFor` with `prebuilt_lym` (`1112-1128`).
  - `changedGoFiles` for a file present on one side only (`redundant.go:427-431`).
  - `WriteOverlay` leaving unreplaced a file whose mutants all dropped (`mutant.go:1565-1579`).
  - `UnbuildableMutants` for a mutant spanning several lines (`mutant.go:1650`).
  - `ProfileBlocks` failure, timeout and missing-profile paths (`runner.go:235-244`).
  - `BuildMutantBinary` returning a plain error with no compile line (`runner.go:286`).
  - Exit code 128 plus the signal number for `lym gate redundant` (`redundant.go:369-372`).

I numbered findings F1 to F6 and F8; F7 was not used.

Tool calls:
1. Read `internal/gatecli/testmap_integration_test.go:655-720` and the base version's `redundant` lines, to check what the task-worktree test asserts on `lym gate redundant` after its fixture dropped to one test. My context held only the diff hunk.
2. Grepped `contracts/specs/refusal-spec.md` for exemptions, to check whether the new refusals are exempt from needing a reaching test. That spec was not in the manifest.
