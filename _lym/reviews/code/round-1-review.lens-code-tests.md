```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Per behaviour, the test that pins it**

- Refusal outside the prime, naming the prime worktree: pinned by `TestGateTestMap/Refusals/OutsideThePrime` (`testmap_integration_test.go:294`).
- `--max < 1` refusal: pinned by `Refusals/MaxBelowOne`.
- No exit record written on a refusal: pinned by the `ReadExitRecord` check after `Refusals` (`:314`).
- Refusal outside any hub: none.
- Refusal when no mapped tier is declared: none.
- Refusal for a named package that is not a package directory: none.
- Stale because no map exists: pinned by `CapLeavesStalePackagesUnbuilt`.
- A fresh map is not rebuilt: pinned by `SecondCallMapsTheRestAndKeepsAFailureAtOneHead`, which checks alpha's `BuiltAt` is unchanged.
- Stale because the map is past the age limit: pinned by `HeadMoveRetriesTheFailedBuildAndAgeRebuilds` (notests aged 8 days).
- Stale because the Go file set differs: none.
- Stale because the content of a Go file differs: none.
- A failed-build map is stale once HEAD moves: pinned by `HeadMoveRetries…`, because the second call's `moved` and `steady` tests commit.
- A failed-build map is not rebuilt at the same HEAD: pinned by the second call (`len(got.Failed) == 0`).
- Dirty skip, covering an untracked file: pinned by `dirtySkip` in every call. The row uses an untracked file only.
- Locked skip: pinned by `lockedSkip`.
- Dirty and locked skips do not count against `--max`: pinned by the `--max 2` call, which builds alpha, fails broken, and still skips both before `Unbuilt: 3`.
- Unbuilt count and the cap: pinned by `Unbuilt: 3`.
- Package path order: only implicit, through the `--max 2` call building alpha then broken.
- Mid-package change discards the map and reports the package skipped: pinned by `movedSkip`, and by `ReadMap(moved)` finding nothing.
- The mid-package skip counts against `--max`: none. Every call that reaches `moved` has room under the cap.
- A landing that leaves the package's hashes unchanged keeps the map, recorded at the HEAD read before the package: pinned by `steady` (`:356`).
- A timed-out test is left out of the map and the package's other tests are still mapped: pinned by `alpha.TestAlphaHangs` in `timed_out` and the `wantTests` list.
- A test in an undeclared tier, or one with no `Require`, is left out: pinned by `TestAlphaTier2` and `TestAlphaUntiered` being absent from the map.
- `RunAlone` kills a timed-out test: pinned by `runner_test.go` `TestRunAlone/TimedOut`.
- `RunAlone` within a non-zero timeout: pinned by `PassWithinTheTimeout`.
- Grace bound when a descendant holds the output pipe: none. The stand-in sleeps directly and spawns no child.
- Build timeout for `go test -c` (`goOutput` timeout message, `BuildTimeout` kill): none.
- `buildLym` timeout counts as failed: none.
- `lym` built once per call, and the `PrebuiltLymEnv` hand-off to `prebuilt_lym` tiers: none. No fixture tier sets `prebuilt_lym`.
- Process-group kill added in `prebuildLym`: none.
- A package with no test file or no mapped test gets a no-tests map: pinned by `notests`, which has `BuildFailure == ""`.
- A coverage build failure gets a map naming the failure, listed in `failed`: pinned by `broken` and `wantBrokenFailed`.
- A `ScanTestFuncs` error leading to `writeFailed`: none.
- Exit record with the pid and the unbuilt count, zero included: pinned by `wantExitRecord(3)` and `(0)`, and by `TestWriteExitRecordReadExitRecord`.
- Exit record written on a failure or cancel after the prime check: none. A `close` regression on those paths would pass.
- `WriteExitRecord` keeping undeclared keys, as its doc says: none.
- Slot held for the whole call and released: pinned only implicitly, by `slots: 1` across five calls.
- The inherited-slot path in `holdSlot`: none.
- Config keys, defaults and the below-1 refusal: pinned by `templateDefaults` and the three new `TestLoadConfig` rows.
- `CodeDirAt` and `CodeFileAt`, covering the commit against the working tree, an absent directory and an absent file: pinned by `TestCodeReadsAtCommit`.
- `CodeDirAt` and `CodeFileAt` error paths: none.
- `hubgeom.IsPrime` and `PrimeWorktreePath`: pinned only through `OutsideThePrime` and the prime-path calls. No unit test, and no error path.
- `modulePackages` exclusions (testdata, dot and underscore directories, nested module): none. Only the flat fixture is walked.
- Named package given as `./alpha`, rebuilt even when fresh: pinned by `NamedPackageIsRebuiltWhenFresh`.
- `gateEnv` refactor in `run.go`: pinned only by the existing `lym gate test` tests, which I did not re-read.
- `ScanTestFuncs`: `testfuncs_test.go` (65 lines) is in the diff, but I did not read it, so I make no claim.

**Findings**

1. MEDIUM, scope, `testmap.go:449`. Staleness from a changed Go file set or changed file content has no test row. A broken `maps.Equal(m.Files, files)` would still pass, because the fresh-map check in the second call only proves unchanged files stay fresh. Fix: add a call after a package's file is committed, or its map's `Files` edited, and assert it is rebuilt.
2. MEDIUM, scope, `runner.go:171`, `testmappackage.go:146`. The build timeout has no test. Neither the `BuildBinary` timed-out error nor the `buildLym` timeout counting as failed is covered. The stand-in build could hang on a stand-in import path, with `minuteUnit` shortened.
3. MEDIUM, scope, `runner.go:105-110`, `runner_test.go:104`. The grace bound with a descendant holding the output pipe has no test. The acceptance text says a descendant must not stall the verb, yet the hang stand-in spawns no child. Without a grandchild, `WaitDelay` and group kill are not exercised beyond the direct child.
4. LOW, scope, `testmappackage.go:82`. The mid-package-discard package counting against `--max` has no row. No call has the cap tight around `moved`.
5. LOW, scope, `testmappackage.go:63`, `testmap.go:139`. The once-per-call `lym` build is not tested. Neither is the failed-build-kept path, nor the `PrebuiltLymEnv` hand-off.
6. LOW, scope, `testmap.go:474`. The exit record on error and cancel paths after the prime check has no test. Only the success path and the refusals are covered.
7. LOW, scope, `testmap.go:162,254,284`, `testmap.go:307`. The outside-any-hub refusal, the no-mapped-tier refusal, the bad named package refusal and the `modulePackages` exclusions have no rows. Cheap as extra rows in the `Refusals` table.
8. LOW, consistency, `testmap_integration_test.go:190`, `PATTERN-test-speed` "Time is injectable". The test shortens the timeout to a real 5 seconds and waits it out twice, in the first call and in `NamedPackageIsRebuiltWhenFresh`. Fix: use a shorter `secondUnit`, for example 50ms.

**Tool calls:** none. My context already held every file I needed.
