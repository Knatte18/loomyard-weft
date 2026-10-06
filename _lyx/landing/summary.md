# Prune the test suites of every package outside the six core ones

Shrinks the Go test suite in every package outside `internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli`, `internal/shedadapters` and `internal/testkit/...`, which `test-suite-prune-core` owns.
Only test code changes, apart from one comment in `internal/stencil/stencil.go` that named tests this change folds.
Across the in-scope packages the top-level test count goes from 3377 to 1997, summed median wall time from 335.32 s to 110.33 s, and summed serial time from 350.56 s to 150.11 s.
No package lost statement coverage.

## What changes

- **Deletions.**
  A test the redundancy report marks `Removable: yes` is deleted after its assertions are read against its covering tests.
  An assertion the covering test did not pin (an error message, an output shape, a refusal's way-forward text) moved into the covering test as a row or an extra check before the deletion.
  A `Removable: no` candidate is never deleted.
- **Folds.**
  Groups of tests that differ only in inputs and expected values became table-driven tests with `t.Run` subtests named after the behavior.
  A fold keeps every assertion, and its table calls `t.Parallel` at the top level and per subtest.
- **Scenario tests.**
  Tests of one tier that each built the same expensive fixture (a hub or pair through hubforge, a tmux server, a seeded worktree) became one `...Scenario` test that builds it once and runs the former tests as ordered `t.Run` steps.
  Examples are `TestSpawnScenario`, `TestRunCLI_MergeScenario`, `TestVerifyGate_Scenario`, `TestRun_TeardownScenario`, `TestCopiedRepoScenario` and `TestResolve_AnchorScenario`.
- **Tier move.**
  `internal/standalonegeom/reedgeom_symlink_integration_test.go` became `reedgeom_symlink_test.go`, an untagged file, because its test spawns nothing.
- **Kept candidates.**
  Every candidate the final rerun still lists carries a `//testtiming:keep <reason>` directive directly above its `func Test…` line.
  The reasons are guard self-checks in `cmd/lyx` (a guard that must be shown to fire on an offending input), assertions the covering tests do not pin, assertions received from a deleted test, and tests that a core or testkit file names.
- **Docs.**
  `docs/benchmarks/test-redundancy.md` has its in-scope `## <package>` sections regenerated; its header and core sections are untouched.
  The new `docs/benchmarks/test-suite-prune-rest.md` records per-package test count, wall and serial time, non-test statement coverage and the load average of every run, before and after, and is linked from `docs/benchmarks/running-tests.md`.
  References to renamed tests were updated in `pattern/PATTERN-batten-bookend.md`, `tools/sandbox/SANDBOX-REED-SUITE.md` and `docs/research/linux-portability-survey.md`.

No `Closes #N`: the change resolves no issue.

## Design decisions to know

- **Ownership boundary.**
  Nothing under the six core packages, `internal/testkit/` or `cmd/testtiming` is edited, so this change and `test-suite-prune-core` touch disjoint files.
  A test that a core or testkit file names was kept under its own name.
- **Skill-loading tests are untouched.**
  Tests of `Spec.Skills`, `SkillLoadSequence`, `SkillUnknown`, `DefaultSkillLoadTimeout`, `loadSkill` and `loadSkillsThenPrompt`, their `SkillLoader` fakes and the stub driver scripts in `internal/loomcli` are left to `skills-in-one-turn`; a candidate among them carries a keep directive naming that task.
- **Scenario steps share state by design.**
  A scenario is an ordered sequence of serial steps on one fixture, a step may build on an earlier one and says so in a comment, and a failing step stops the scenario.
  A scenario's top-level test calls `t.Parallel`, and no step does.
  A merge stays within one tier, so an `integration` test never joins a `tmux` scenario and no gate stops running a test it ran before.
  A family was merged when the merge removed fixture builds; no per-family timing was taken, and the per-package wall times in the report are the measurement.
- **Report splice.**
  `cmd/testtiming` at this base takes one go pattern for `-pkg` and rewrites its `-out` file whole, so each in-scope section was spliced into `test-redundancy.md` by hand from a per-package rerun.
- **No tool change.**
  The tool at this base ignores `//testtiming:keep`, so a rerun on this branch still lists the kept candidates.
  They show under "Kept" once `test-suite-prune-core`'s reading of the directive lands.

## To check

- **Wall-time rises.**
  Thirteen packages show a wall delta above zero, each by 0.10 s or less, and the timing runs started at higher load (5.91 to 8.41 on the 1-minute average) than the before runs (3.67 to 4.59).
  `internal/gitkit` (`TestCopiedRepoScenario` replaces nine `CopyRepo` builds with two) and `internal/lyxcwd` (`TestResolve_AnchorScenario` replaces nine checkout copies with one) have a scenario merge behind the rise, since the steps now run serially.
  The others (`internal/buildinfo`, `internal/configsync`, `internal/loomrecipe`, `internal/orchengine`, `internal/parentreview`, `internal/selfreportcli`, `internal/selfreportengine`, `internal/standalonegeom`, `internal/treadleengine`, `internal/weftname`, `tools/internal/devbin`) have no merge and sit within the spread of the three timing runs.
- **Tool artifact.**
  `cmd/lyx` `TestExitCode_HelpPaths` is listed `failed` by the tool, but passes when run alone and was `failed` in the before report as well.
- **Coverage.**
  A block-by-block comparison of the final profile with the before profile finds no lost non-test block, and no drop is recorded as noise.
  The final rerun surfaced two lost blocks, restored by a process-cwd row in `TestRunCLIIn_FromNonGitDirectory` (`internal/configcli`) and a params-differ case in `TestWriteSeed_RefusesDisagreeingSeed` (`internal/shedrun`).
  `internal/reedcli`, `internal/shedrun` and `tools/sandbox` gain coverage points (+1.3, +0.8, +2.0).
- **Deletions and their receivers.**
  Each deleted candidate's unique assertion lives on in its covering test, and the diff is the list.
  Where a deletion left a test with a moved assertion, `TestMark_NeverDoubleWraps` moved into `TestClassAndMark`, and the received assertion is named on the receiver's keep directive where the rerun lists it.
- **Not run by hand.**
  `llm`-tagged tests are out of scope and were not run.
  The `tmux`-tagged scenarios depend on the local tmux, so a reviewer on another machine should run `go test -tags integration,tmux` on the in-scope packages once.
- **No shared fake was missing.**
  No integration test stayed behind for a missing shared fake, so no `// tiermove:` comment was added and nothing is listed for `test-suite-prune-core`.
