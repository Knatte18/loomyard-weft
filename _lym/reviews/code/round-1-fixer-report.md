# Fixer report, code review round 1

## Changes

Each fix is its own commit, except F2, F8 and F10, which share files and one commit, plus a later F2 follow-up.

- F1 fixed as proposed: `internal/gateslot/doc.go` Tier plan table gives `webster-fix`, `code-burler-fix` and `darn-write` scope `coverage`; `pr-review` stays `diff`.
- F2 fixed with a changed test, since the proposed cache row cannot fail.
  `TestCoverageSelections` in `internal/gatecli/coveragescope_test.go` drives `coverageSelections` with no maps directory and asserts each package misses with `testcover.MissNoMap`.
  The proposed "second call returns the cached selection" row is not made: with no maps directory `openSelector` runs once and shares one `setupMiss`, so the assertion holds without the cache.
  The proposed unreadable-maps-directory row is not made: it is unreachable without a hub, since `fabricengine.Open(nil)` dereferences nil.
- F3 fixed with its own change.
  `TestTierNumbers` in `internal/gatecli/tiers_test.go` pins that a tier's own run and its selected run list the tier once, and that vet and guard steps are left out.
  The proposed green rows are not added: the existing `TestGreenRecord` row "a guard-filter entry never covers an empty-args request" and the `newestWholeAncestor` row "a whole entry under args is never a base" already pin that an entry holding `-run` args neither covers an empty-args request nor serves as a base.
  The entry carries the args because `green.go:447` records `Args: run.args`, and the plan rows of `TestPlanTierRuns` pin `-run` in the selected run's args.
- F4 fixed as proposed: a row of `TestGateTest_Scopes` runs `--gate darn-write` over a whole fallback and under the `none` scoper.
- F5 fixed as proposed: `TestSelectorSelect` asserts `Selection.Recorded` names equal the map's tests, and are empty with a miss.
- F6 fixed as proposed: the sample gates block in `internal/gatecli/doc.go` shows `scope: coverage` for the three keys.
- F7 fixed as proposed: `_lym/landing/summary.md` names `fabricengine.CodeWorktreeChanges`. The file is not tracked, so it is not committed.
- F8 fixed as proposed: `importerPackage` labels a guard-scan error with its own text and keeps the parse miss for any other read or parse error.
  A row of `TestNarrowCoverage` covers it.
  The `narrowCoverage` doc comment and the summary name the new label.
- F9 fixed as proposed: two rows of `TestTargetsFor` over a graph with `internal/f`, an importer whose tests build the binary.
- F10 fixed as proposed: `TestNarrowCoverage` rows for an unresolved test, a test with no `Require`, a covering tier, an unparseable test file and a misplaced guard marker; `TestCoverageTier` rows for an undeclared tier and a command with no target placeholder.

Verification: `lym gate test` over `internal/gatecli`, `internal/testcover`, `internal/impactset` and `internal/gateslot` passes at tier 0, and `--tier 1 ./internal/gatecli` passes.
The final change, to `internal/gatecli` alone, passes at tier 0.

A gate finding then flagged a comment line of `internal/impactset/targets_test.go` that reached the line limit.
It is split into two sentences on two lines, in one more commit, `impactset: fix gate finding — split an over-long comment line`.

## Self-check

- fix-code-correctness, MEDIUM, `TestCoverageSelections` passes without the cache: fixed with an own change. The cache assertion is removed, and the F2 entry above says the cache is unpinned.
- fix-proposal, LOW, same finding as above: merged with it.
- fix-proposal, LOW, F4 row does not depend on the key being `coverage`: disputed, every fix worse. See Disputed.
- fix-proposal, LOW, F8 branch unreachable from `computeScope`: not changed. See Disputed.
- fix-consistency, LOW, summary does not name the guard-scan label: fixed.
- fix-consistency, LOW, `narrowCoverage` doc omits the guard-scan case: fixed.
- fix-affected-set, LOW, `docs/dotnet-hub.md` sample shows `diff`: disputed, premise false. See Disputed.

## Disputed

- Self-check, `docs/dotnet-hub.md:119-121`, premise false: the sample is a hand-written hub file with scoper `dotnet`, and the fork's own report states that coverage narrowing applies to the `go` scoper only, so `diff` and `coverage` behave alike there and the sample stays correct.
- Self-check, F4 row independent of `coverage`, every fix worse: the row pins `withGateScope` mapping `coverage` to the computed form and that a whole fallback prints no ` coverage: ` line, which is what F4 asked for. Under `none` and under a fallback the behaviour equals `diff` by design (D1), so no row can tell them apart there.
- Self-check, F8 branch unreachable from `computeScope`, every fix worse: `DeriveTargets` scans every package first, so only a file changed between the two scans reaches the branch. The review's `fix` asks for the label, and the branch is harmless and tested through `narrowCoverage`. Removing it restores the wrong label.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this round.
