---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/impactset/targets.go:45
    summary: Removing the coverage scope leaves ModuleTargets.Importers, BinaryBuilders and splitUnchanged computed on every diff-scope derivation with no production reader
    fix: In internal/impactset/targets.go delete the Importers and BinaryBuilders fields of ModuleTargets (lines 45-50) and their assignments at lines 147-152, keeping the Packages assignment. In internal/impactset/graph.go delete splitUnchanged (lines 148-167) and packageArgsOf (lines 169 on) if nothing else calls it. In internal/impactset/doc.go delete lines 36-38, which describe the two fields and the narrowing caller. Fold or delete the internal/impactset/targets_test.go row that sets modules[0].Importers (line 23) so the test asserts only Packages.
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: design
    location: internal/gatecli/testmappackage.go:236
    summary: An ahead coverage build that will never be used is waited for in full instead of cancelled, in takeBuild for a mismatched package and in close after a HEAD move, cap or error
    fix: In internal/gatecli/testmappackage.go give coverageBuild a cancel func and run each startBuild under its own context.WithCancel(r.ctx), passing that context as the Toolchain's Ctx. In takeBuild, when build.pkg != pkg, call build.cancel() before waiting on build.done and removing the binary. In internal/gatecli/testmap.go close (line 609), call r.ahead.cancel() before waiting on r.ahead.done. Update the takeBuild and close doc comments to say an unused ahead build is cancelled.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/gatecli/testmappackage.go:185
    summary: No test pins that the next package's coverage build starts while the current package's tests run, or that an unused ahead build is ended when the call ends
    fix: In internal/gatecli/testmap_integration_test.go add a row to an existing multi-package subtest of TestGateTestMap, not a new top-level test. Have the stand-in go record a timestamped line for each coverage build start, using the testMapCoverBuildsEnv recorder, and have a stand-in test of the first package record its start and end. Assert that the second package's build-start line falls between the first package's test start and end. If F2 is fixed, add a row where the walk ends after the first package, for example through a HEAD move, and assert the call returns before a hanging ahead build's timeout.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: internal/gatecli/doc.go:1050
    summary: The doc justifies running every test side by side by each process having its own working directory, but every run starts in the same package directory
    fix: "In internal/gatecli/doc.go line 1050 replace the reason with what holds: each test runs in its own process with its own environment and profile, all in the package directory, so a test that writes a fixed path there can disturb another. A failure or timeout beside others is bounded by the re-run alone, but a run that still passes is recorded as it ran."
    origin: lens:code-correctness
---

### [LOW] Coverage-scope support in impactset left with no production reader

**Location:** `internal/impactset/targets.go:45-50`, `:147-152`; `internal/impactset/graph.go:148-172`; `internal/impactset/doc.go:36-38`

**Issue:** `ModuleTargets.Importers` and `BinaryBuilders`, and the `splitUnchanged` split that fills them, came in with cdc3522a ("gate: add a coverage scope that narrows importing packages' tests").
The deleted `coveragescope.go` was their only production reader.
A grep of `.Importers` and `.BinaryBuilders` now finds only `internal/impactset/targets_test.go:23`.
They are still computed on every `diff` derivation, and `doc.go:38` still says a caller narrows the importers.
The board entry removes "the gate's map-based selection", and this is the part of it the diff leaves behind.
The test-only-code guard cannot see it, because `splitUnchanged` is still reached.

### [LOW] Ahead build waited for in full instead of cancelled

**Location:** `internal/gatecli/testmappackage.go:236-249` (`takeBuild`), `internal/gatecli/testmap.go:609` (`close`)

**Issue:** `startBuild` runs under `r.ctx` (`testmappackage.go:216-217`).
Job i+1 can return before `takeBuild`: its lock may be held, it may have failed at HEAD, its map may have moved, or it may have no mapped test.
Then job i+2's `takeBuild` waits for job i+1's whole `go list` and build before dropping it.
After a HEAD move, the cap or an error ends the walk, `close` likewise waits for the ahead build, which can take up to `redundancy_build_timeout_min`.
The results stay correct, but the call wastes build time and exits late.

### [LOW] Ahead-build overlap has no test

**Location:** `internal/gatecli/testmappackage.go:185-187`, `:208-249`

**Issue:** Design item 6 asks for the test binaries to be built ahead.
Deleting the ahead start at `:185-187` would leave every test green: the stand-in records builds as lines only (`testmap_integration_test.go:418-425`), with no ordering against test runs, and no subtest checks for one.
`SecondCallMapsTheRestAndKeepsAFailureAtOneHead` reaches the dropped-build path indirectly, but nothing asserts the overlap itself.

### [LOW] Side-by-side justification claims a separate working directory per process

**Location:** `internal/gatecli/doc.go:1050`

**Issue:** The line reads "since each process has its own environment and working directory".
`runTest` starts every run with `RunAlone(p.bin, p.pkgDir, …)`, so all runs share the package directory.
The smart-build report this design follows says that tests side by side may fail or record differently, and that the re-run alone bounds a failure but not a changed record.
The doc's reason hides that bound.

## Rejected

- code-tests finding 2 (no row makes `KeyBlocks`, `CoveredFuncs` or `ProfiledFuncs` fail inside `reduce`): this failure path is not new. Before the diff, the same key and naming failures went from `fillMapTests` and `selfCheck` to `writeFailed`. The diff moves the path; it does not add a behavior, so the rubric's uncovered-behavior bullet does not apply.
- Handler check, design item 7 not taken: the miss-causes report makes no recommendation that soundly lowers the packages a build re-measures. It says its name-following estimate "bounds nothing" and would need type information. Its bookkeeping finding is the lean loop the diff implements. The summary states this reason, and it holds.
- Handler check, all queued tests run side by side whatever their `t.Parallel`: the smart-build report's step 2 (lines 271-275) asks for exactly this, with a re-run alone on failure.

## Notes

- `lym board get testmap-on-command` failed with a JSON error, so the entry was judged from the decision record's `## Goal` body.
- The redundancy report's contained candidates: `TestReconcileAll`'s changed row pins the prune of both retired beam keys, which no covering test asserts. `TestRun_ABeamTheHoldReleasedSchedulesAgain` and `TestRun_NoPassWhileAMoveStands` only lost the upkeep counters; their assertions and coverage are otherwise unchanged by this diff. No fold is raised on them.
- The timed full build is a one-run measurement report, set beside the report's estimate as the entry asks.
