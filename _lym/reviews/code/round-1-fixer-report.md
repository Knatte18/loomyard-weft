# Fixer-report — code review round 1

Every finding is fixed.
The fixes are in one commit, not one per finding, because they share files and the tests of several span the same subtests.
The self-check changes follow in a second commit.
`lym gate test --tier 1 ./internal/testcover ./internal/gatecli` and the tier-0 run of both packages pass.

## Changes

- **F1** (fixed as proposed, test reduced): `testcover.Toolchain` has a `Dir`, which `goOutput` sets as `cmd.Dir`.
  `mapPackage` sets it to `r.moduleDir`.
  The stand-in `go test -c` now fails unless its cwd holds `go.mod`; the process cwd (`internal/gatecli`) holds none, so the test fails without the fix.
  The proposed case of a tier dir below the worktree root is not added: it needs a second fixture layout, and the cwd check already pins that the build runs in the resolved module root.
- **F2** (fixed as proposed): `Toolchain` has a `Ctx`, the parent of the command's timeout context, set from `r.ctx` in `mapPackage` and `runTest`.
  `mapPackage` returns `r.ctx.Err()` instead of calling `writeFailed` when a coverage build or the `lym` build ends with the call context done, and after the test loop.
  `buildLym` returns the context error, leaves `lymBuilt` false and keeps no `lymErr` then.
  No test sends a signal, since the finding's fix asked for none.
- **F3** (fixed as proposed): subtest `ChangedFileContentAndFileSetRebuild` edits and commits a file of `alpha` and adds one to `steady`, then asserts both are built with the new hashes.
- **F4** (fixed as proposed): `TestRunAlone` row `ReturnsWithinTheGraceWhileADescendantHoldsTheOutput`, with a stand-in test whose child inherits stdout and sleeps past the grace.
  `TestBuildBinaryTimesOut` covers the `BuildTimeout` error.
  Subtest `BuildPastItsTimeoutFailsThePackage` covers the timeout branch of the package build, listing the package as failed.
- **F5** (fixed as proposed): `prebuildLym` takes the pipe wait and sets `cmd.WaitDelay`, treating `exec.ErrWaitDelay` as a successful build.
  The tier run passes its `pipeWaitDelay`, testmap its `grace`.
- **F5 extra, not proposed:** `prebuildLym` also takes an `env`; nil inherits the process's, as before.
  testmap passes the slot's environment (`r.holderEnv`), as its coverage builds already run under it.
  The prebuilt-`lym` subtest needs it: the build otherwise ignores the call's environment, so the test's stand-in `go` is never chosen.
- **F6** (fixed as proposed): `mapPackages` skips a named package with no tracked Go file, cause `no Go file tracked at HEAD`.
  Subtest `NamedPackageWithNoTrackedGoFileIsSkipped` reaches it through a git-excluded package.
- **F7** (fixed as proposed, one variant): `Refusals` has rows `OutsideEveryHub`, `NoMappedTier` and `UnknownPackage`.
  Its exit-record assertion now expects the record written by the refusals that come after the call's state exists.
  Subtest `SkippedPackageCountsAgainstMax` runs `--max 2` over `broken` and `moved`, with a later stale `notests` counted unbuilt; the review's `--max 1` would leave `moved` unbuilt before it is skipped.
  Subtest `PrebuiltLymIsBuiltOncePerCall` asserts one `lym` build and one path seen by every run of two packages.
  The fixture gains `testdata`, a dot directory and a nested module, which the exact `Built` lists would show if walked.
- **F8** (fixed as proposed): `doc.go` says "the pipe wait the gate already uses".
- **F9** (fixed with a different value): `secondUnit` is 150ms, so the 20-second test timeout is 3 seconds instead of 5, not the proposed 50ms; see the self-check.

## Gate repair

The comment gate flagged five comments added by this round: fixed-column wrapped breaks and over-long lines in
`internal/gatecli/doc.go`, `internal/gatecli/prebuild.go`, `internal/gatecli/testmap_integration_test.go`,
`internal/testcover/doc.go` and `internal/testcover/runner.go`.
Each is rewritten as one sentence per line; no wording changed beyond the split.
A third commit holds the repair.
The gate then flagged one comment of that repair still wrapped, the variables comment in `testmap_integration_test.go`, now split into two sentences in a fourth commit.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: `doc.go` and the `testmap` help miss the new skip cause — fixed.
- fix-consistency, LOW: `testcover/doc.go` misses `Dir` and `Ctx` — fixed.
- fix-consistency, LOW: `mapPackage` comment misses the cancelled-call error — fixed.
- fix-consistency, LOW: the stand-in's doc comment misses its `lym` build role and the seen-path record — fixed.
- fix-consistency, LOW: the `prebuild.go` header names only the tier run — fixed.
- fix-consistency, LOW: `gateYAML` comment checked, no change needed — no change.
- fix-proposal, MEDIUM: a 1-second timeout (50ms unit) can time out a passing stand-in test on a loaded machine — fixed with own change: 150ms unit, 3 seconds, long enough to start and still short for the hang rows.
- fix-proposal, no finding: the `env` parameter is noted as an unrequested change, not a defect.

## Disputed

None.

## Deferred

None.

## Rulings

None reached me.
