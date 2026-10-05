# Investigate why the test suite runs several times slower than its August benchmark

`test-suite-regression` · Note · enhancement

The test suite has slowed several-fold since the 2026-08-13 benchmark (`docs/benchmarks/test-suite-timing.md`).
It is not a correctness bug, but most of the slowdown is avoidable.

## Measured 2026-10-05 (Ryzen, idle, `-count=1`, warm cache, default `-p`)

| | Benchmark commit `16c0cfcc8`, rerun today | HEAD `a0dae77c3` |
| --- | --- | --- |
| Untagged | 3.4 s | 8.8 s |
| Integration | 17.9 s | 57.5 s |
| Compile/link only (`-run XXX_none`) | 1.9 s | ~5.0 s |

The comparison is like for like: same command, same machine, old commit re-measured.
Growth since then: 1959 → 5749 test functions, 63 → 114 test packages, plus the tree-sitter cgo grammars;
growth and cgo explain the compile floor and roughly half the rest.

## Culprits

- `internal/fabricengine` sets the integration floor (16.7 s → 54.5 s): 698 tests, 425 of them serial (44 s summed).
  Whole files lack `t.Parallel`, e.g. `mergein_recovery_integration_test.go`, `commit_integration_test.go`, `pull_integration_test.go`, `warpbinding_clone_integration_test.go`.
- `internal/reedcli` (under 1 s → 49 s) waits on real production constants.
  `watchdog_integration_test.go` `TestWatchdogIntegration_ExitsAfterIdleCyclesAndReleasesLock` waits three 5 s discovery cycles (`watchdog.go`; from `d39b30648`).
  `watchdogreap_integration_test.go` waits out `reapExitTimeout` of 15 s (`reedengine/lifecycle.go`; from `aa5fcc61a`).
  The resize test takes 5.4 s (`d39b30648`).
- `internal/verifyrun/run_integration_test.go` `TestRun_Cancellation` waits out the 10 s `waitDelay` const (`verifyrun/run.go`; from `021c67023`).
- `internal/verifytree/verify_integration_test.go` runs a real `sleep 5` where `true` would do (`11ce5b003`).
- `internal/lyxcwd/enforcement_test.go` `TestEnforcement_FabricVocabulary` sets the untagged floor (5.1 s): 85% of its CPU is `vocabExemptPattern.ReplaceAllString`, run on every AST node (`3ae398a30`).

The real-time-wait lever regressed: the waits above have no test seam.
The hermetic git env is intact except in `gitrepo/internal/gitoracle`.

## Fixes

1. Give the watchdog interval, reap timeout and verifyrun `waitDelay` a test seam, and use `true` in verifytree; integration should drop by roughly 35–45 s.
2. Add `t.Parallel` to fabricengine's serial integration files where they are independent.
3. In `bareVocabularyTokenIn`, skip the regex when the lowercased text contains neither "weft" nor "warp".
4. Accept the ~5 s compile/link floor.
5. Record the new measurement and slowest-tests table in `docs/benchmarks/test-suite-timing.md`, and consider a guard test against new production-constant waits.

Related: `test-tiers` (which tests each gate runs); together they form one task.
