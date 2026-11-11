## Fixer-report, code review round 1

Every finding was fixed.
Gates: `lym gate test ./internal/testcover` (tier0) and `lym gate test --tier 1 ./internal/gatecli` pass; tier0 over both packages passed before the follow-up commit, which touched `internal/testcover` and the one integration test.

## Changes

- **F1** (`internal/gatecli/testmapstamp_test.go`, commit `9f14b8ee`):
  `TestStampEnvironment` has rows for an unchanged input, a changed tier command, a changed tier env value, a changed tier `parallel` (tiers digest equal) and a changed hub env variable (hub env digest differs), and asserts the toolchain fields are kept.
  `TestMappedTierConfigs` pins that only declared mapped tiers are returned.
  The test sits in `package gatecli` with its siblings, so no `export_test.go` was needed.
- **F2** (`internal/testcover/declcompare.go`, `testselect.go`, `testselect_test.go`):
  `compareTestSources` no longer misses on imports and reports `addedImports` and `removedImports` in `testFileComparison`.
  `packageChanges` judges them.
  Fixed differently from the proposal (see the Self-check): an added import is accepted when it is the package's own, linked, or standard library; a removed one is accepted when it is the package's own or standard library only, as `compareGoSources` treats a deleted file's imports.
  A removed import of a linked package misses, because it can drop an `init` from the test binary.
  Rows: new test file importing `testing` (fixture changed), removed test file importing `testing`, standard-library import added, import outside the linked set added (the old row also changed a package-level var, so it never isolated the import), removed linked import, external test file importing the package.
  The fixture's fake GOROOT gained `testing`.
- **F3** (`testselect_test.go`): rows for GOOS, GOARCH, tiers digest and hub env digest; a `wantDetail` field asserts the exact miss detail on the stamp rows and the build-failure row.
- **F4** (`testselect.go`, `testselect_test.go`): `mapMiss` misses on `BuildFailure` only for a map with no test entries.
  Rows: `BuildFailureWithNoTestEntries` (miss) and `BuildFailureBesideTestEntries` (no miss).
- **F5** (`testselect_test.go`): `TestSelectorSelect_UnreadableTarget` gives an unknown commit as target and asserts an error and no miss.
  A commit unknown to the fake store fails at the target's `go.mod` read, which is the `environmentMiss` path the finding names; the fake store cannot fail one file of a known commit, so the `readSides` path stays unpinned.
- **F6** (`internal/gatecli/coveragescope_unix_integration_test.go`, tier1, `!windows`):
  `TestOpenSelector_ToolchainEnvironmentUnreadable` runs `openSelector` over a hubforge hub with a stand-in `go` script that answers `env GOROOT` and fails every other call, and asserts no selector and a `MissMapUnreadable` miss whose detail holds `go env: `.
  The finding's proposed fix, a `goBinary` that fails, would fail earlier at the GOROOT read, so the stand-in splits the two calls.
- **F7** (`testselect_test.go`): row `OwnTestPackageClauseChanged`, `MissTestFile`.

Commits: `9f14b8ee` (F1), `d7b407dc` (F6), `6e420e59` (F2, F3, F4, F5, F7, one commit for the shared test file), and a follow-up commit for the self-check changes.

## Disputed

None.

## Deferred

- The other readers of `Map.BuildFailure` keep their rule: `internal/gatecli/redundancy.go:305` treats any non-empty value as a failed map and `internal/gatecli/testmap.go:459` rebuilds on it.
  Reason: no writer produces a failure record beside test entries (`writeFailed` clears `Tests`), so nothing breaks today, and which rule is right depends on what D5's upkeep plan writes, which the parent holds.
  The `BuildFailure` field doc in `internal/testcover/coveragemap.go` is likewise unchanged.
  The parent should align these once D5's field is settled.

## Rulings

None.

## Self-check

- fix-code-correctness, LOW, go env assertion could pass through the GOROOT failure: fixed as proposed (`"go env: "`).
- fix-code-correctness, LOW, external test file importing its own package missed: fixed (own import accepted, row added).
- fix-consistency, LOW, `BuildFailure` field doc and `MissBuildFailure` text: deferred with the `BuildFailure` item above.
- fix-consistency, LOW, `redundancy.go` twin of the rule: deferred, same item.
- fix-proposal, MEDIUM, removed linked import accepted for a test file: fixed with own change (standard library only; row added).
- fix-proposal, LOW, in-module imports outside the linked set still miss: not changed.
  `fillMapTests`' doc and `mapPackage` record every in-module package the test binary links in `Linked`, so a test-only import such as `tiergate` is in the set of a map built with that test file; a file new since the map can import a package the map's binary never linked, and that miss is the intended one.
- fix-affected-set, LOW, same `BuildFailure` readers: deferred, same item.
- All four forks returned a report.
