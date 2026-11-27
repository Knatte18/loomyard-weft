```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens review: what each guard, refusal and test row in the plan can actually reach.

**Findings**

- **BLOCKING** (Card 3, the `ATestTheMemoryGuardKillsIsNamedAndTheRunGoesOn` subtest, D4): the guard this row needs never runs, so the row cannot pass as written.
  - `guardOwnTests` starts the guard only when `memoryGuardApplies` finds no `lym gate test` ancestor named by `enclosingGateEnv` (`internal/gatecli/memoryguard.go`, `guardOwnTests` and the lines above it).
  - Every tier1 test runs under `lym gate test`, which sets that variable to its own pid, and that process is an ancestor of the test. So inside the test the map run starts no guard and prints no kill line.
  - The enclosing gate's own guard does sample the tree, but against the real hub's `memory_quit_mb`, which is far above `testMapHoardMegabytes` (384). So `TestGuardedHoldsMemory` passes, and neither the failed entry nor the stderr line appears.
  - The row needs a seam instead, for example a `testMapRequest` field for the guard's starter, as `testRequest.memoryGuardFor` does for tier runs.

- **BLOCKING** (Card 4, D6): a map write the card does not list still happens unchecked.
  - In `testMapRun.mapPackage`, when every mapped test is spawn-marked, the `len(toRun) == 0` branch calls `r.write(pkg, m)` with no build (`internal/gatecli/testmappackage.go:145-146`).
  - Card 4 lists checks for "a package with no mapped test", for a failed test or spawn scan, for after the build, and for a failed listing. It does not list this branch.
  - So a package whose tests are all spawn-marked gets its map written over dirty files or a moved HEAD. D6 says a package with no build gets one check before its write.

- **MEDIUM** (Card 4, D6): a failed-measurement write the card does not list also happens unchecked.
  - In `testMapRun.measureTests`, an error from `runBatch` (a reduction failure from `packageRun.reduce`, such as `KeyBlocks` or `CoveredFuncs`) goes to `r.writeFailed(p.job.pkg, p.m, err)` before `endJob` runs (`internal/gatecli/testmappackage.go:278-284`).
  - Card 4 adds `checkClean` only in `endJob`. So that failure write skips the check before the write, and could record a failure over a dirty tree or a moved HEAD.

- **LOW** (Card 1, the package set, D1/D6): dropping the `planPackage` guard lets an ignored package get a map.
  - The removed guard skipped a package that tracks no Go file at HEAD (`testmapplan.go:33-37`).
  - Card 1's set takes a package with "a tier0 or tier1 test (`mappedTests`)". `mappedTests` reads the files on disk (`ScanTestFuncs`), not HEAD.
  - A gitignored package directory holding test files enters the set. `fileHashes` at HEAD is empty for it, and Card 4's status checks cannot see an ignored file.
  - So the package gets a map with empty `Files`. The D6 bound covers a gitignored Go file inside a package, not a whole untracked package. The fixture's `ignored` package has no test file, so no row exercises this.

**Tool calls**
- Read `detachfront.go` 257-380 (`cappedCall`, `testRequest.words`): to check whether a detached `--map` call reaches `flagShapeRefusal`. My context held only the agent's summary.
- Grepped the `gateslot` config validation for `memory_mb`, `memory_quit_mb` and `redundancy_test_memory_mb`, and read `Pool.AcquireSlots`: to check that the `MemoryBelowTheTestCap` input is accepted by the config and how a count of 0 behaves. My context did not hold either.
- Read `memoryguard.go` around `guardOwnTests`, and grepped `clihelp.GroupRunE`: to check when the guard runs and the exact unknown-subcommand text. My context did not hold them.
- Grepped `TestGateTest_RefusesBeforeResolvingTheHub`'s entry point and `impactset.DeriveTargets`'s fallbacks: to check that the refusal rows run through the CLI and that the `--base` row's scope can come back narrow. My context did not hold either.
