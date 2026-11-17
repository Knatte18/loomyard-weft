```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests under the code-tests lens (code-tests). For each behaviour the diff adds or changes, name the test that pins it, or none.

**What each behaviour is pinned by**

- **Capping and kill reporting.**
  - The scope cap ends a child over the cap and reports the kernel's kill: `TestRunMemoryCapped_EndsAChildOverTheCapByTheHostsMeans/AScopeReportsTheKernelsKill`. It is skipped where the probe fails.
  - The fallback with no user manager fails the child's own allocation and reports no kill: same test, row `WithNoUserManagerTheChildFailsItsOwnAllocation`.
  - A child within the cap ends on its own and one over it is ended, on every platform: `TestRunMemoryCapped`.
  - A cancel's group kill still reaches a capped run and is no cap kill: `TestRunMemoryCapped_ACancelEndsTheRunAndIsNoCapKill`.
  - The probe's `cgroup.subtree_control` check (`probeMemoryScope`): none. Each host exercises only one branch, and a wrong answer only switches which row is skipped.
- **Memory limit reading and the clamp.**
  - `ReadMemoryLimit`: high/max minimum, file cache subtracted, tightest level wins, levels skipped, not found: `TestReadMemoryLimitAt`.
  - The headroom ÷ cap clamp, the at-least-one rule and parallel 1 when the cap covers each process alone: `TestClampParallelToMemory`.
  - The wiring in `memoryCappedParallel` (probe, then `ReadMemoryLimit`, then clamp): none. `TestGateRedundant` replaces it with a stub.
- **Windows** `JOB_OBJECT_LIMIT_JOB_MEMORY` and the job terminate on cancel: none have run. `TestRunMemoryCapped` would cover them on Windows, and the summary admits it has not run there.
- **`testcover.RunAlone`.**
  - A run under `MemoryLimit` passes with its arguments and profile unchanged: `TestRunAlone/PassUnderTheMemoryLimit`.
  - A run over the cap comes back failed with `MemoryCapped`, not an error: `TestRunAlone/MemoryCapped`. On a host where the probe passes, this row reaches only the kernel-kill branch.
  - The `outOfMemoryLine` detection: pinned only on a host where the probe fails (see finding 3).
  - `ProfileBlocks` under `MemoryLimit`: none, but it reuses `runTestBinary`, which `TestRunAlone` pins.
- **`gate redundant`.**
  - A capped mutant run counts as a kill and is listed in `memory_capped`: `TestGateRedundant`, through the `memory_capped` diff and the `Hoard` candidates' `MutantsKilled: 1`.
  - A capped run is never rerun alone: `TestGateRedundant`'s run-count assertion (`- cappedCoveringRuns`).
  - Timed-out mutants are listed in `timed_out`: `TestGateRedundant`'s `timed_out` diff.
  - The mutant is logged at spawn: none (a log line, not asserted).
  - The timeout floor drops from 10 s to 2 s: none directly. The `timed_out` assertion would pass at either floor.
  - A capped unmutated run sets `FailsUnmutated`: none specific. It falls out of the existing "did not pass" path.
- **`gateslot`.**
  - `redundancy_test_memory_mb` defaults to 2048: `templateDefaults` in `internal/gateslot/config_test.go:72`.
  - A zero value is refused: `TestLoadConfig/redundancy_test_memory_mb zero`.
- **The other callers.**
  - The redundancy report step lists a capped test as not judged, with `reasonMemoryCapped`: none.
  - `gate testmap` passes `MemoryLimit` and records a capped run as failed: none.
  - `cmd/testtiming -redundancy` gets a 2 min / 2048 MB bound: none.

**Findings**

1. MEDIUM · design · `internal/gatecli/redundancy.go:553` and `internal/gatecli/testmappackage.go:672`
   No test pins the capped path of the redundancy report step (the `result.MemoryCapped` case that returns `reasonMemoryCapped`), nor testmap's `MemoryLimit` wiring. Dropping either `MemoryLimit: …` assignment, or the new `case`, fails no test.
   Evidence: the grep for `reasonMemoryCapped|MemoryCapped|testMemory` across `_test.go` files finds no hit in a testmap or redundancy-report test.
2. LOW · design · `cmd/testtiming/redundancy.go:68`
   No test pins the new time and memory bounds on `measurePackage`'s runner, the change the board entry asks for in "give it the same bounds".
   Evidence: the same grep finds no `cmd/testtiming` hit.
3. MEDIUM · design · `internal/testcover/runner.go:157` (`outOfMemoryLine`) and `:838`
   `TestRunAlone/MemoryCapped` pins only one branch per host: the kernel kill where the scope probe passes, the out-of-memory regex where it fails. On a host like Neve the regex branch is never run through `RunAlone`. That branch is the only route by which the fallback and Windows paths set `MemoryCapped`.
   Evidence: `internal/testcover/runner_test.go:250` has a single `MemoryCapped` row with no way to force the fallback, unlike proc's `RunMemoryCappedWithoutScope`.
4. LOW · design · `internal/gatecli/redundantmutants.go:404` (`mutantTimeoutFloor = 2`)
   No assertion pins the floor drop. The `timed_out` list in `TestGateRedundant` would be the same at a 10-unit floor.
   Evidence: `internal/gatecli/redundant_integration_test.go:310-313`.
5. LOW · design · `internal/gatecli/redundantmutants.go:663-671` (`memoryCappedParallel`)
   The production clamp wiring is replaced by a stub in `TestGateRedundant`'s request (line 313). Mixed-up arguments to `clampParallelToMemory`, such as `limited` and `coversTree`, fail no test.

Tool calls:
- grep of `_test.go` files for `MemoryCapped`, `reasonMemoryCapped`, `memory_mb`, `MemoryLimit` and `testMemory`: to find which tests cover the capped paths in the report step, testmap and testtiming, which my context did not show.
